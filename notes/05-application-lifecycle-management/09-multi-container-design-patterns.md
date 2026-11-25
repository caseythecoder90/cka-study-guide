# 09 — Multi-Container Design Patterns

Three ways to run a helper next to the main app ([08](08-multi-container-pods.md)). They differ in **when the helper starts and when it stops**.

![Three timelines. Co-located: both containers start with no guaranteed order and run until the Pod ends. Regular init: each init container runs to completion in order, then the app starts. Sidecar: the sidecar starts first, the app runs alongside it, the app stops, then the sidecar stops](./diagrams/04-container-patterns.svg)

| | Co-located | Regular init container | Sidecar |
|---|---|---|---|
| Declared under | `containers` | `initContainers` | `initContainers` + `restartPolicy: Always` |
| Starts | With the app, order not guaranteed | Before the app, one at a time | Before the app |
| App waits for it to | Nothing | **Exit 0** | **Start** (and pass its `startupProbe`, if it has one) |
| Runs | For the Pod's life | Until done | For the Pod's life |
| Stops | No guaranteed order | Before the app starts | **After** the app |
| Counted in READY | Yes | No | Yes |
| Probes | Yes | No | Yes |
| Use for | Two peers with no ordering need | Wait for a database, prepare files | Log shipper, proxy |

---

## 1. The YAML

A Pod with both kinds of init container. Runnable version: [`examples/multi-container/init-and-sidecar.yaml`](../../examples/multi-container/init-and-sidecar.yaml).

```yaml
spec:
  initContainers:
  - name: log-shipper            # sidecar: started first, keeps running
    image: busybox:1.36
    command: ["sh", "-c", "tail -n+1 -F /var/log/app/app.log"]
    restartPolicy: Always        # the ONLY difference from a regular init container
    volumeMounts:
    - {name: logs, mountPath: /var/log/app}
  - name: wait-for-db            # regular: must exit 0 before the app starts
    image: busybox:1.36
    command: ["sh", "-c", "until nc -z db 5432; do echo waiting; sleep 2; done"]
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "while true; do date >> /var/log/app/app.log; sleep 5; done"]
    volumeMounts:
    - {name: logs, mountPath: /var/log/app}
  volumes:
  - name: logs
    emptyDir: {}
```

- **`initContainers` run in list order.** A regular one must finish before the next starts. A sidecar only has to start.
- **`initContainers` sits beside `containers`**, at the same indentation. Its position in the file doesn't matter.
- **The two `restartPolicy` fields are different.** The one on the init container accepts `Always` here and makes it a sidecar. The one on `spec` is the Pod's.
- The slides write `command: 'script.sh'`. A plain string is rejected: `command` is a list ([03](03-commands-and-arguments-kubernetes.md)).

## 2. When an init container fails

- The kubelet **retries that container** until it succeeds, with back-off. The app never starts in the meantime.
- If the Pod's `restartPolicy` is `Never`, the whole Pod becomes `Failed` instead.
- Init containers run **again** if the Pod is restarted or replaced, so the work must be safe to repeat.

| STATUS | Meaning |
|---|---|
| `Init:0/2` | 0 of 2 init containers are done. Stuck here: the current one is still running or waiting |
| `Init:Error` / `Init:CrashLoopBackOff` | An init container exits non-zero |
| `PodInitializing` | Init containers are done, and the app containers are starting |

```bash
k describe pod app                   # "Init Containers:" section: State, Exit Code, Reason
k logs app -c wait-for-db            # init containers need -c
k get pod app -o jsonpath='{.spec.initContainers[*].name}'
```

## 3. What restarts when a container fails

**Only the container that failed.** The kubelet replaces that one container inside the same Pod. The other containers keep running, and the Pod keeps its name, UID, IP and volumes. `spec.restartPolicy` is written once at Pod level, but it is **applied to each container separately**:

| Pod `restartPolicy` | App container exits 0 | App container exits non-zero | Regular init exits non-zero | Sidecar exits (any code) |
|---|---|---|---|---|
| `Always` (default) | Restarted | Restarted | Retried | Restarted |
| `OnFailure` | Left stopped | Restarted | Retried | Restarted |
| `Never` | Left stopped | Left stopped | **Pod `Failed`** | Restarted |

So for an app with a logging agent:

| Event | Logging agent | App | Pod |
|---|---|---|---|
| Agent crashes | Restarted, with back-off | **Keeps running, not restarted** | Same Pod, **not Ready** until the agent is back |
| App crashes | Keeps running | Restarted | Same Pod, not Ready until the app is back |
| Pod deleted, evicted, or its node fails | Gone | Gone | A controller creates a **new** Pod. A bare Pod is not replaced |

**The catch is readiness.** A Pod is Ready only when **all** its app containers and sidecars are. An agent stuck in `CrashLoopBackOff` shows as `1/2`, and the Pod is removed from its Service's endpoints. The app never restarts, but it stops receiving Service traffic.

The course article contradicts itself here. Its statements that "the entire Pod is restarted" when a main container fails, and that a failed init container restarts the Pod and "all init containers are rerun", are wrong by default:
- A failed init container is retried **by itself**. Those that already succeeded are not rerun.
- All of them run again only if the Pod itself is restarted or replaced, for example when its sandbox is recreated.
- Restarting every container is opt-in: a `restartPolicyRules` entry with the action `RestartAllContainers` (beta since 1.36). You won't need it for the exam.

```bash
k get pod app                        # RESTARTS is the total across containers
k get pod app -o jsonpath='{range .status.containerStatuses[*]}{.name}{"\t"}{.restartCount}{"\t"}{.ready}{"\n"}{end}'
k get pod app -o jsonpath='{.metadata.uid}'    # unchanged = same Pod, only a container restarted
```

Sidecars are listed under `.status.initContainerStatuses` instead.

## 4. Log shipping with a sidecar

The course example is Filebeat, a log shipper, sending to **Elasticsearch** (stores and searches logs), with **Kibana** as the dashboard. It's the same shape as the Fluent Bit example in [08](08-multi-container-pods.md). As a sidecar the shipper is already running when the app writes its first line, and it outlives the app, so it can also ship the lines that explain a crash or shutdown.

Both share the Pod's termination grace period (30 seconds by default). If the app uses all of it, the sidecar gets little or no time to flush.

In a **Job**, a co-located helper that never exits keeps the Job from completing. A sidecar doesn't.

---

## Exam angle

- **"Add an init container that waits for / creates X":** add it under `initContainers`, and check that the Pod goes from `Init:0/1` to `Running`.
- **"Add a sidecar that streams the log file":** `initContainers` with `restartPolicy: Always`, plus a shared `emptyDir` mounted in both. If the task only says to add a container, a regular one under `containers` is enough. Verify with `k logs <pod> -c <sidecar>`.
- **Pod at `1/2` with a restart count:** find which container with `describe`. The other one was not restarted.
- **Pod stuck in `Init:...`:** run `describe`, then `logs -c <init container>`. Common causes are a typo in its command, or the Service it waits for doesn't exist.
- **Traps:**
  - leaving out `restartPolicy: Always` on a long-running helper, so the app never starts;
  - listing a regular init container that needs the sidecar **before** the sidecar;
  - adding an init container to a live Pod, which needs `replace --force`.
- **Recall:** `kubectl explain pod.spec.initContainers.restartPolicy`.

## References

- [Init containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) — "Init containers in use" (a Pod that waits for Services) and "Detailed behavior"
- [Sidecar containers](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) — the log-shipper Deployment YAML, and differences from init and app containers
- [Pod lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/) — "Pod-level container restart policy" (the table of exit codes) and "Pod conditions" (`ContainersReady`)
- [Debug init containers](https://kubernetes.io/docs/tasks/debug/debug-application/debug-init-containers/) — the `Init:` statuses

## Related notes

- CKAD: [Design patterns](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/03-multi-container-pods/02-design-patterns.md) — the developer's view
- CKAD: [Init containers](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/03-multi-container-pods/03-init-containers.md) — the developer's view
