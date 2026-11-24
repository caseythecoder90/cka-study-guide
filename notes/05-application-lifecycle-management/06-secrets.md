# 06 — Secrets

A **Secret** is a ConfigMap ([05](05-configmaps.md)) for sensitive values: passwords, tokens, keys, certificates. It is created the same way and consumed the same ways, as environment variables or as files. Two things are different:
- values under `data` are **base64-encoded**;
- the kubelet keeps mounted Secret files in memory (tmpfs) on the node, never on its disk.

**Base64 is an encoding, not encryption.** Anyone who can `get` the Secret can decode it. By default it is also stored unencrypted in etcd. Protecting Secrets is covered in section 4.

A runnable version of sections 2 and 3 is in [`examples/secrets/secret-volume.yaml`](../../examples/secrets/secret-volume.yaml).

---

## 1. Creating one

```bash
k create secret generic app-secret --from-literal=DB_User=root --from-literal=DB_Password=paswrd
k create secret generic app-secret --from-file=app_secret.properties   # one key = whole file
k create secret generic app-secret --from-env-file=app.env             # one key per KEY=value line
k create secret tls web-tls --cert=tls.crt --key=tls.key               # keys tls.crt and tls.key
k create secret docker-registry regcred --docker-server=<reg> --docker-username=<u> --docker-password=<p>
k create secret generic app-secret --from-literal=DB_Password=paswrd $do -o yaml   # see the encoded YAML
```

`kubectl` does the base64 encoding. The `--from-*` flags behave exactly as for ConfigMaps.

| `kubectl create secret` | `type:` | Used for |
|---|---|---|
| `generic` | `Opaque` | Anything |
| `tls` | `kubernetes.io/tls` | A certificate and key, e.g. for an Ingress |
| `docker-registry` | `kubernetes.io/dockerconfigjson` | Pulling from a private registry, via `imagePullSecrets` |

## 2. Writing the YAML: `data` or `stringData`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:                         # values MUST be base64
  DB_Host: bXlzcWw=           # echo -n 'mysql' | base64
  DB_User: cm9vdA==           # echo -n 'root'  | base64
stringData:                   # plain text; the API server encodes it into data
  DB_Password: paswrd
```

- **Encode with `echo -n`.** Plain `echo` adds a newline, which becomes part of the password. The app then fails to log in, and the cause is invisible in `describe`.
- **Long values:** GNU `base64` wraps its output at 76 characters. Use `base64 -w0`.
- **Decode with `base64 -d`** (same as `--decode`).
- **`stringData` is write-only.** The API server encodes it into `data`, so `get -o yaml` only ever shows `data`. If a key is in both, `stringData` wins.
- A plain value under `data`, such as `DB_User: root`, is rejected as illegal base64 (or decodes to garbage).

## 3. Using it in a Pod

As environment variables (details in [04](04-environment-variables.md)):

```yaml
    env:
    - name: DB_Password
      valueFrom:
        secretKeyRef:            # one key
          name: app-secret
          key: DB_Password
    envFrom:
    - secretRef:                 # every key
        name: app-secret
```

As files: one file per key, with the **decoded** value as content.

```yaml
spec:
  containers:
  - name: app
    image: busybox
    volumeMounts:
    - name: app-secret-volume
      mountPath: /opt/app-secret-volumes
      readOnly: true
  volumes:
  - name: app-secret-volume
    secret:
      secretName: app-secret     # secretName, NOT name (configMap uses name)
      defaultMode: 0400          # optional; the default is 0644
```

`ls /opt/app-secret-volumes` shows `DB_Host DB_Password DB_User`, and `cat .../DB_Password` prints `paswrd`. Everything else matches ConfigMap volumes ([05](05-configmaps.md), sections 3 and 4): `items` picks keys, the mount hides the directory, mounted files update within about a minute, and env vars and `subPath` files never update.

## 4. How safe are they?

- **Stored unencrypted in etcd by default.** Anyone with etcd access or an etcd backup can read them. Fix: encryption at rest, configured on the kube-apiserver with `--encryption-provider-config` ([07](07-encrypting-secrets-at-rest.md)).
- **RBAC is the real protection.** Grant `get`/`list` on Secrets narrowly. Anyone who can create a Pod in a namespace can also mount, and so read, every Secret in that namespace.
- **Never commit Secret manifests to git.** The values are only encoded. Generate them at deploy time, or keep them in an external store such as Vault or a cloud secrets manager.
- **The [Secrets Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/)** mounts values from an external store straight into the Pod as a CSI volume. The secret then never has to exist as a Kubernetes Secret in etcd. A `SecretProviderClass` object says which store and which keys to fetch. This is not on the CKA, but it is common in production.
- **What Kubernetes does protect:** a Secret is only sent to nodes running a Pod that uses it, and the kubelet keeps it in tmpfs and deletes it when the Pod goes.

## 5. Checking

```bash
k get secrets                                    # TYPE and DATA (number of keys)
k describe secret app-secret                     # key names and sizes in bytes; values hidden
k get secret app-secret -o yaml                  # encoded values
k get secret app-secret -o jsonpath='{.data.DB_Password}' | base64 -d   # one value, decoded
k exec <pod> -- cat /opt/app-secret-volumes/DB_Password
```

The slides contain a few errors:
- The Pod manifest has `lablels:` at the wrong level. `labels` belongs under `metadata`.
- The `envFrom` slide references `app-config`, where it should reference `app-secret`.
- `describe` shows `DB_Host: 10 bytes`, but `mysql` is 5 bytes.
- The `default-token-xxxxx` Secret in the `get` output is no longer created automatically since Kubernetes 1.24. Pods now get short-lived projected ServiceAccount tokens instead.

---

## Exam angle

- **"Create Secret X with these values":** use `k create secret generic X --from-literal=...` and let kubectl encode the values.
- **"Expose Secret X to Pod Y as env / at /path":** use `secretKeyRef`/`secretRef`, or a `secret` volume with `secretName`. Verify with `k exec Y -- env` or `k exec Y -- ls /path`.
- **"What is the password in Secret X?":** run `-o jsonpath='{.data.KEY}' | base64 -d`, and write the decoded value, not the encoded one.
- **Pod stuck in `CreateContainerConfigError` or `ContainerCreating`:** the Secret or key is missing, or the Secret is in another namespace. `describe pod` names it.
- **Traps:**
  - `echo` without `-n`;
  - plain text under `data`;
  - `name:` instead of `secretName:` in the volume;
  - submitting the base64 string when asked for the value.

## References

- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) — "Types of Secret", "Using a Secret" (env and volume YAML), "Immutable Secrets"
- [Managing Secrets using kubectl](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/) — `create secret generic` forms and "Decode the Secret"
- [Distribute credentials securely using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/) — a Pod that mounts a Secret as a volume or reads it as env
- [Good practices for Kubernetes Secrets](https://kubernetes.io/docs/concepts/security/secrets-good-practices/) — encryption at rest, least-privilege RBAC

## Related notes

- CKAD: [Secrets](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/05-secrets.md) — the developer's view
