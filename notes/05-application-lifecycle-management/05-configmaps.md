# 05 — ConfigMaps

A **ConfigMap** stores non-secret configuration as key-value pairs under `data`, outside the image. A Pod consumes it in one of two ways:
- as **environment variables**: `env` or `envFrom` ([04](04-environment-variables.md));
- as **files**, through a volume.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

A runnable version of sections 2 and 3 is in [`examples/configmaps/configmap-files.yaml`](../../examples/configmaps/configmap-files.yaml).

---

## 1. Creating one

```bash
k create cm app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
k create cm app-config --from-file=app.properties          # ONE key "app.properties" = whole file
k create cm app-config --from-file=settings=app.properties # same, key renamed to "settings"
k create cm app-config --from-file=config-dir/             # one key per file in the directory
k create cm app-config --from-env-file=app.env             # one key PER LINE (KEY=value)
k create cm app-config --from-file=app.properties $do > cm.yaml   # see the YAML it builds
```

**`--from-file` versus `--from-env-file`** is the difference that matters:
- `--from-file` keeps the file whole as a **single key**. Use it for a config file the app reads.
- `--from-env-file` splits `KEY=value` lines into **separate keys**. Use it for variables you want to load with `envFrom`.

## 2. A whole file as one key

A key's value can be an entire config file. In YAML, write it as a **block scalar** after `|`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-config
data:
  LOG_LEVEL: info                  # a plain key...
  app.properties: |                # ...and a key whose value is a whole file
    server.port=8080
    cache.enabled=true
  nginx.conf: |
    server {
      listen 80;
      location / { proxy_pass http://backend:8080; }
    }
  agent.yaml: |                    # YAML inside YAML: just a string to Kubernetes
    output:
      host: ${OUTPUT_HOST}         # NOT filled in by Kubernetes
      port: ${OUTPUT_PORT}
```

- **`|`** keeps the line breaks and ends with one newline. **`|-`** strips the final newline. For config files either works. `|-` matters for single-line values that must not end in `\n`.
- **The file's lines are indented one level deeper than the key**, and that indentation is removed from the stored value. A wrong indent ends the block early, and the next line is parsed as another key or rejected.
- **Kubernetes treats the content as an opaque string.** It never parses or validates the YAML or properties inside, and it doesn't fill in `${VAR}` placeholders. If they get filled in, **the app** does it, from environment variables you set on the container. Logging agents such as Fluent Bit work this way. Kubernetes' own `$(VAR)` expansion only happens in `command`, `args` and `env` values, never in ConfigMap data.
- Key names may contain letters, digits, `-`, `_` and `.`, so `app.properties` is a valid key. It is also the file name you get when the ConfigMap is mounted.

## 3. Mounting as files

![A ConfigMap with keys APP_COLOR and app.properties, mounted three ways: the whole ConfigMap at /etc/config gives one file per key and updates live but hides the directory's existing contents; items picks and renames keys; subPath places one file into an existing directory without hiding it, but never updates](./diagrams/02-configmap-volume-mounts.svg)

**Every key becomes a file named after the key, and the file's content is the key's value.** A key holding a whole config file therefore turns back into that file, and a plain key such as `APP_COLOR` becomes a one-line file.

```yaml
spec:
  containers:
  - name: web
    image: nginx
    volumeMounts:
    - name: config                     # 1. whole ConfigMap: one file per key
      mountPath: /etc/config
    - name: config-items               # 2. only some keys, renamed
      mountPath: /etc/app
    - name: config                     # 3. one key as one file, into an existing directory
      mountPath: /etc/nginx/conf.d/default.conf
      subPath: nginx.conf
  volumes:
  - name: config
    configMap:
      name: web-config
  - name: config-items
    configMap:
      name: web-config
      items:
      - key: app.properties
        path: settings.properties      # file name inside mountPath
```

| Method | What you get | Live updates | Watch out |
|---|---|---|---|
| Whole ConfigMap | One file per key in `mountPath` | Yes, after about a minute | **Hides everything already in that directory.** Mounting at `/etc/nginx` would hide `nginx.conf` |
| `items` | Only the listed keys, at the paths you choose | Yes | Keys not listed are left out. A listed key that doesn't exist stops the Pod starting |
| `subPath` | One file, the rest of the directory untouched | **No**: restart the Pod | `mountPath` is the full file path |

`ls -la` in a mounted directory shows the files are symlinks into a hidden `..data` directory. That's how the kubelet swaps in new content in one step.

## 4. Updating

- **Env vars:** fixed at container start. Restart with `kubectl rollout restart deploy/<name>`.
- **Mounted files:** refreshed by the kubelet within about a minute. The app still has to re-read them, and many only read config at startup.
- **subPath files:** never refreshed.
- **To change a ConfigMap:** `k edit cm app-config`, or regenerate it and apply: `k create cm app-config --from-file=app.properties $do | k apply -f -`.
- **`immutable: true`** (a top-level field, next to `data`) blocks changes to the data. To change it, delete and recreate the ConfigMap.

## 5. Checking

```bash
k get cm                                         # DATA = number of keys
k describe cm app-config                         # every key and value
k get cm app-config -o yaml
k get cm app-config -o jsonpath='{.data.app\.properties}'   # one key; escape the dot
k exec <pod> -- ls /etc/config ; k exec <pod> -- cat /etc/config/app.properties
```

---

## Exam angle

- **"Create ConfigMap X with these values":** use `--from-literal` repeatedly, or `--from-file` if a file is given. Check `--from-file` versus `--from-env-file` against what the task expects the keys to be.
- **"Mount ConfigMap X at /path in Pod Y":** add one `configMap` volume plus one `volumeMount`. Verify with `k exec Y -- ls /path`.
- **"Put this config file into the container without breaking the directory":** use `subPath`, with `mountPath` set to the full file path.
- **Pod stuck in `ContainerCreating` or `CreateContainerConfigError`:** the ConfigMap or a listed key is missing, or it's in another namespace. `describe pod` says which.
- **Traps:**
  - mounting over a directory the app needs;
  - expecting env vars or `subPath` files to update;
  - wrong indentation under `|`.

## References

- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) — "ConfigMaps and Pods" (a ConfigMap with file-like keys), "Using ConfigMaps as files from a Pod", "Mounted ConfigMaps are updated automatically" (with the note that `subPath` mounts never update), "Immutable ConfigMaps"
- [Configure a Pod to use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) — every `kubectl create configmap` form, "Add ConfigMap data to a Volume", and "Add ConfigMap data to a specific path in the Volume" (`items`)
- [Volumes: using subPath](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath) — `subPath` YAML for mounting one file

## Related notes

- CKAD: [ConfigMaps](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/04-configmaps.md) — the developer's view
