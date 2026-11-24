# 08 — Multi-Container Pods

A Pod can run several containers that belong together: one main app plus helpers that are developed and released separately but must run next to **every** instance of the app. Scale the Deployment and each new replica gets its own copy of every container.

Containers in one Pod share three things:

| Shared | What it means | The fine print |
|---|---|---|
| **Lifecycle** | Scheduled to the same node together, and deleted together | **One container crashing restarts only that container**, not the Pod |
| **Network** | One IP and one network namespace: containers reach each other on `localhost:<port>` | Two containers can't listen on the same port |
| **Storage** | Volumes are declared once, at Pod level | A container only sees a volume it **mounts** in its own `volumeMounts` |

The course says the containers are "created together and destroyed together". That's true of the Pod, but the containers start in list order without waiting for each other. Ordering takes init containers or native sidecars (section 4).

---

## 1. Writing one

There is no imperative flag for a second container. Generate one and add to the `containers` list:

```bash
k run web --image=nginx $do > pod.yaml     # then add a second "- name:" entry under containers
```

```yaml
spec:
  containers:
  - name: web-app
    image: nginx
    ports:
    - containerPort: 8080
  - name: main-app              # every container needs a unique name
    image: busybox
    command: ["sleep", "3600"]
```

## 2. Working with a running one

```bash
k get pod web                              # READY 2/2: ready containers / total containers
k logs web -c main-app                     # -c picks the container
k logs web --all-containers                # all of them, one after another
k exec -it web -c main-app -- sh
k get pod web -o jsonpath='{.spec.containers[*].name}'
k describe pod web                         # per-container State, Restart Count, Last State
```

Without `-c`, `logs` and `exec` pick the first container (or the one named in the `kubectl.kubernetes.io/default-container` annotation) and print which one they chose.

**Troubleshooting order for `1/2 READY` or `CrashLoopBackOff`:**
1. Run `describe pod`, and find **which** container is unready or restarting.
2. Run `logs -c <that one>`, and add `--previous` for the crash before the restart.
3. Check that container's own `command`/`args`, `env`, mounts and probes.

The healthy container is a distraction.

## 3. A real example: app plus log-shipper sidecar

A common production layout: the app writes log **files**, and a Fluent Bit sidecar tails them and ships them to the log platform (Splunk, Elasticsearch, an OpenTelemetry collector). The runnable version is [`examples/multi-container/app-with-log-shipper.yaml`](../../examples/multi-container/app-with-log-shipper.yaml).

![The orders Pod: the orders-api container writes app.log to a shared emptyDir volume, the log-shipper container mounts the same volume read-only, tails the file and ships the records out. Both load the orders-runtime ConfigMap with envFrom; the shipper's fluent-bit.yaml and parsers.yaml come from a second ConfigMap through subPath mounts](./diagrams/03-app-log-shipper-pod.svg)

```yaml
    spec:
      volumes:
      - name: app-logs
        emptyDir: {}                       # shared scratch space, deleted with the Pod
      - name: log-shipper-conf
        configMap: {name: log-shipper-conf}
      containers:
      - name: orders-api                   # the application
        image: <app-image>
        envFrom: [{configMapRef: {name: orders-runtime}}]
        volumeMounts:
        - {name: app-logs, mountPath: /app/logs}           # writes app.log here
      - name: log-shipper                  # the sidecar
        image: fluent/fluent-bit:4.2.0
        args: ["-c", "/fluent-bit/etc/fluent-bit.yaml"]
        envFrom: [{configMapRef: {name: orders-runtime}}]  # the same settings as the app
        env:
        - name: POD_NAME
          valueFrom: {fieldRef: {fieldPath: metadata.name}}
        volumeMounts:
        - {name: app-logs, mountPath: /app/logs, readOnly: true}   # reads the app's files
        - {name: log-shipper-conf, mountPath: /fluent-bit/etc/fluent-bit.yaml, subPath: fluent-bit.yaml}
        - {name: log-shipper-conf, mountPath: /fluent-bit/etc/parsers.yaml, subPath: parsers.yaml}
```

How each piece works, and which chapter covers it:

- **The shared `emptyDir` is the whole trick.** Both containers mount the same volume, so the app's files appear in the sidecar. Neither needs a network call or a PersistentVolume. The volume lives as long as the Pod, which is fine because the logs are shipped within seconds.
- **`args` without `command`** ([03](03-commands-and-arguments-kubernetes.md)): the Fluent Bit image's entrypoint is the `fluent-bit` binary, so `args` only adds `-c <config>`. Here `-c` is Fluent Bit's config flag; it has nothing to do with `sh -c`.
- **`subPath` mounts** ([05](05-configmaps.md)): each key of the ConfigMap becomes one file in `/fluent-bit/etc`, and the image's other files there stay visible. The catch: editing the ConfigMap changes nothing until the Pod restarts (`k rollout restart`).
- **Shared `envFrom`** ([04](04-environment-variables.md)): one ConfigMap gives both containers the same app name and environment. Values that differ per container go in each container's own `env`.
- **The downward API (`fieldRef`)** gives the sidecar its Pod name, namespace and node, so every shipped record says where it came from. Fluent Bit replaces `${POD_NAME}` in its config with the environment variable. Kubernetes doesn't.
- **Everything else is per container:** `resources`, `securityContext` (for example `runAsUser`), `ports` and probes. **The Pod's resource request is the sum of all its containers' requests,** and the scheduler places the Pod on that total. A sidecar's requests count too.
- **Probes stay on the app.** A startup or liveness probe on `orders-api` restarts only `orders-api`. The shipper keeps running and picks the file up again.

The alternative is to log to **stdout** and let a node-level DaemonSet collect `/var/log/containers` ([04-02](../04-logging-and-monitoring/02-managing-application-logs.md)). It's simpler, with no sidecar per Pod, but less flexible per app. Teams use a file-tailing sidecar when the app can only write files, or when each app needs its own parsing and enrichment.

## 4. Startup and shutdown order

Regular containers start in the order listed, **without** waiting for each other, and stop in no guaranteed order. With a log shipper, that means the first or last log lines can be missed. Two fields control the order:

- **`initContainers`** run to completion, one at a time, **before** any app container starts. Use them for setup work. The next lecture covers them.
- **Native sidecars:** an init container with `restartPolicy: Always`. It starts **before** the app containers and keeps running. It is stopped **after** them on shutdown, and it doesn't block a Job from completing. Enabled by default since Kubernetes 1.29; stable since 1.33. Moving `log-shipper` here is the modern fix for the ordering gap:

```yaml
  initContainers:
  - name: log-shipper
    image: fluent/fluent-bit:4.2.0
    restartPolicy: Always          # this one field makes it a sidecar
```

---

## Exam angle

- **"Add a sidecar container named X to Pod/Deployment Y":**
  - For a Pod: `k get pod Y -o yaml > y.yaml`, add the container, then `k replace --force -f y.yaml`. **The containers list of a running Pod can't be edited.**
  - For a Deployment: `k edit deploy Y`, which starts a rollout.
- **"Share logs between containers":** use an `emptyDir` volume, mounted in **both** containers. Verify with `k exec Y -c X -- ls /path` or `k logs Y -c X`.
- **Fastest generation:** `k run x --image=a $do > x.yaml`, then copy the container block and change `name`/`image`.
- **Traps:**
  - forgetting `-c`;
  - duplicate container names;
  - mounting the volume in only one container;
  - two containers on one port;
  - debugging the healthy container instead of the one `describe` flags.

## References

- [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) — "Pods with multiple containers" and "Resource sharing and communication" (shared network and storage)
- [Sidecar containers](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) — `restartPolicy: Always` in `initContainers`, startup and shutdown order
- [Logging architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/) — "Sidecar container with a logging agent" (this chapter's pattern) versus the node-level agent
- [Communicate between containers in the same Pod using a shared volume](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/) — the `emptyDir` YAML

## Related notes

- CKAD: [Multi-container Pods](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/03-multi-container-pods/01-multi-container-pods-intro.md) — the developer's view
- CKAD: [Design patterns](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/03-multi-container-pods/02-design-patterns.md) — sidecar, ambassador, adapter
