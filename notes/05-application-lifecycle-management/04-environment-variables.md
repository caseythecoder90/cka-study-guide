# 04 — Environment Variables

`docker run -e APP_COLOR=pink img` becomes `env` on the container in a Pod spec. Each entry takes its value from one of four places:

| Source | Field | Use for |
|---|---|---|
| A literal | `value` | Fixed settings |
| A ConfigMap key | `valueFrom.configMapKeyRef` | Settings shared or changed outside the Pod |
| A Secret key | `valueFrom.secretKeyRef` | Passwords, tokens |
| The Pod itself | `valueFrom.fieldRef` / `resourceFieldRef` | Pod name, namespace, IP, node, limits (the *downward API*) |

`envFrom` imports **every key** of a ConfigMap or Secret at once. ConfigMaps and Secrets get their own chapters next. This one is about getting their values into a container.

Runnable versions of these manifests are in [`examples/env-vars/`](../../examples/env-vars/).

---

## 1. One variable at a time: `env`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: webapp
    image: nginx
    env:
    - name: APP_COLOR                 # literal
      value: pink
    - name: APP_MODE                  # one key from a ConfigMap
      valueFrom:
        configMapKeyRef:
          name: app-config            # ConfigMap name
          key: APP_MODE               # key inside it
    - name: DB_PASSWORD               # one key from a Secret
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_PASSWORD
    - name: POD_NAME                  # downward API
      valueFrom:
        fieldRef:
          fieldPath: metadata.name    # also: metadata.namespace, status.podIP, spec.nodeName
    - name: MEM_LIMIT
      valueFrom:
        resourceFieldRef:
          containerName: webapp
          resource: limits.memory
```

- `value` must be a **string**, so quote numbers and booleans: `value: "8080"`, `value: "true"`.
- The variable's `name` and the `key` don't have to match. `name` is what the app sees; `key` is what the ConfigMap or Secret stores.

## 2. All keys at once: `envFrom`

```yaml
    envFrom:
    - configMapRef:
        name: app-config              # every key becomes a variable
    - secretRef:
        name: app-secret
    - prefix: DB_                     # optional: DB_HOST, DB_PORT, ...
      configMapRef:
        name: db-config
```

`envFrom` is a list, like `env`. Each item names **one** source.

If the same variable appears more than once, `env` wins over `envFrom`. Among `envFrom` sources, the later one wins.

## 3. Behavior that matters

- **Variables are fixed when the container starts.** Editing the ConfigMap or Secret later does **not** change the variables of running containers. Restart them: `kubectl rollout restart deploy/<name>`. Mounted volumes do update; env doesn't.
- **A missing ConfigMap, Secret or key** stops the container from starting: the status is `CreateContainerConfigError`, and `kubectl describe pod` says which one is missing. Add `optional: true` under the `configMapKeyRef`/`secretKeyRef` (or under the `envFrom` item) to start anyway.
- **They're in the same namespace.** A Pod can only reference ConfigMaps and Secrets in its own namespace.
- **`env` can't be changed on a running Pod.** Use `kubectl replace --force` ([03](03-commands-and-arguments-kubernetes.md), section 4) or edit the Deployment template.
- **Variables can build on each other with `$(VAR)`.** `value: "http://$(HOST):$(PORT)"` works if `HOST` and `PORT` are defined **earlier** in the same `env` list.

## 4. Imperative

```bash
k run webapp --image=nginx --env=APP_COLOR=pink --env=APP_MODE=prod $do    # literals only
k set env deploy/webapp APP_COLOR=blue                     # literal; triggers a rollout
k set env deploy/webapp --from=configmap/app-config        # copies every key, as configMapKeyRef entries
k set env deploy/webapp --from=secret/app-secret --keys=DB_PASSWORD
k set env deploy/webapp --list                             # what is set
k exec webapp -- env | grep APP_                           # verify inside the container
```

`kubectl run` can only set literals. For `valueFrom` or `envFrom`, generate the YAML with `$do` and add the block. `kubectl explain pod.spec.containers.env.valueFrom` recalls the field names.

---

## Exam angle

- **"Set variable X on Pod/Deployment Y from ConfigMap Z, key K":**
  - write the `env` + `configMapKeyRef` block, or run `k set env deploy/Y --from=configmap/Z --keys=K` (it reuses the key name as the variable name);
  - verify with `k exec <pod> -- env`.
- **"Load all keys of ConfigMap/Secret X":** use `envFrom` with `configMapRef` or `secretRef`.
- **Pod stuck in `CreateContainerConfigError`:** `describe` names the missing ConfigMap, Secret or key. Check the name, the key and the **namespace**.
- **Traps:**
  - `configMapKeyRef` vs `configMapRef`: KeyRef sits under `env.valueFrom`, Ref sits under `envFrom`.
  - unquoted numbers in `value`;
  - expecting running Pods to see an updated ConfigMap.

## References

- [Define environment variables for a container](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/) — literal `env`, and `$(VAR)` references between variables
- [Configure a Pod to use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) — "Define container environment variables using ConfigMap data" (`configMapKeyRef`) and "Configure all key-value pairs in a ConfigMap as container environment variables" (`envFrom`)
- [Distribute credentials securely using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/) — "Define container environment variables using Secret data" (`secretKeyRef`, `envFrom` with `secretRef`)
- [Expose Pod information through environment variables](https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/) — `fieldRef` and `resourceFieldRef`

## Related notes

- CKAD: [Environment variables](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/03-environment-variables.md) — the developer's view
