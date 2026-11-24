# 07 — Encrypting Secret Data at Rest

By default the kube-apiserver writes Secrets to etcd **in plain text**: base64 only exists in the API, and etcd holds the raw value ([06](06-secrets.md), section 4). Anyone with an etcd client certificate, or a copy of a snapshot, can read every password in the cluster.

**Encryption at rest** makes the kube-apiserver encrypt chosen resource types before writing them to etcd and decrypt them on read. Clients see no change: `kubectl get secret -o yaml` still shows base64. Only what is stored in etcd changes.

It is configured with one file, an `EncryptionConfiguration`, passed to the kube-apiserver with `--encryption-provider-config`.

---

## 1. Is it on already?

```bash
grep encryption-provider-config /etc/kubernetes/manifests/kube-apiserver.yaml   # kubeadm cluster
ps -ef | grep kube-apiserver | grep -o 'encryption-provider-config=[^ ]*'      # any cluster
```

No output means encryption at rest is off. To prove it, read a Secret straight from etcd (etcd flags in [02-04](../02-core-concepts/04-etcd-in-kubernetes.md)):

```bash
k create secret generic secret1 --from-literal=mykey=mydata
ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/secret1 | hexdump -C
```

`mydata` is readable in the hexdump: the Secret is stored unencrypted.

## 2. The configuration file

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets                       # which resource types to encrypt
  providers:                      # ORDER MATTERS
  - aescbc:                       # first provider: used to ENCRYPT every new write
      keys:
      - name: key1
        secret: <32 random bytes, base64>   # head -c 32 /dev/urandom | base64
  - identity: {}                  # fallback: still READ data written unencrypted
```

**Provider order is the whole trick:**
- **The first provider encrypts** everything written from now on.
- **On read, every provider is tried in order** until one can decrypt.
- **If `identity: {}` is first, nothing is encrypted.** Identity means "store as is". A config file that lists identity first looks like encryption but does nothing.
- Keep `identity` **last**, so Secrets written before encryption was turned on can still be read.

| Provider | Notes from the docs |
|---|---|
| `identity` | No encryption. The default |
| `aescbc` | Used in most tutorials and labs. Not recommended for production (vulnerable to padding-oracle attacks) |
| `aesgcm` | Fast, but the key must be rotated every 200,000 writes |
| `secretbox` | Strong and fast |
| `kms` (v2) | Envelope encryption: the key lives in an external KMS (cloud KMS, Vault). **The production choice** |

With `aescbc`, `aesgcm` and `secretbox`, the key sits in a plain file on the control-plane node. That protects etcd and its backups, but not someone who already has root on the node. KMS fixes that by keeping the key off the node.

`kubectl explain` doesn't cover this file. The YAML is on the docs page under "Write an encryption configuration file".

## 3. Turning it on (kubeadm cluster)

The kube-apiserver is a **static Pod**: a container that can't see the node's files unless they are mounted into it. Three edits to `/etc/kubernetes/manifests/kube-apiserver.yaml`:

```bash
mkdir -p /etc/kubernetes/enc
vi /etc/kubernetes/enc/enc.yaml          # the file from section 2
```

```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml   # 1. the flag
    volumeMounts:
    - name: enc                                                   # 2. mount into the container
      mountPath: /etc/kubernetes/enc
      readOnly: true
  volumes:
  - name: enc                                                     # 3. the node directory
    hostPath:
      path: /etc/kubernetes/enc
      type: DirectoryOrCreate
```

Saving the manifest makes the kubelet restart the kube-apiserver, which takes up to about a minute. Check that it comes back:

```bash
watch crictl ps | grep kube-apiserver      # kubectl fails until it is back up
k get pods -n kube-system
```

If it doesn't come back, the most likely causes are a typo in the config file or a missing mount. Read the logs on the node: `crictl ps -a | grep apiserver`, `crictl logs <id>`, or `ls /var/log/pods/kube-system_kube-apiserver*`.

## 4. Existing Secrets are not encrypted

Turning encryption on only affects **new writes**. Secrets that already exist stay in plain text until they are written again. Rewrite all of them:

```bash
k get secrets -A -o json | k replace -f -
```

Then verify with the same `etcdctl get` as in section 1. The value now starts with `k8s:enc:aescbc:v1:key1:`, followed by unreadable bytes. The `-o yaml` output through kubectl is unchanged.

## 5. Rotating or removing the key

- **Rotate:**
  1. Add the new key as the **second** entry under `keys` and restart the API server.
  2. Move the new key to the **first** position and restart again.
  3. Run the `replace` command from section 4.
  4. Remove the old key.
- **Turn encryption off:** put `identity: {}` **first**, keep the old provider after it so existing data can still be read, then run the `replace` command.

On a cluster with several control-plane nodes, every kube-apiserver needs the same file.

---

## Exam angle

- **Probably not a CKA task.** The 2025 CKA curriculum doesn't list encryption at rest, but it is a standard **CKS** task. Still, the skill it exercises is core CKA: editing the kube-apiserver static Pod manifest without breaking it.
- **The task:** "Encrypt Secrets at rest with aescbc, and make sure existing Secrets are encrypted." Write the config file, add the flag plus the volume mount, wait for the API server, run `replace`, and verify with `etcdctl ... | hexdump -C`.
- **Before editing, copy the manifest out of the manifests directory:** `cp kube-apiserver.yaml /root/`. A copy left inside `/etc/kubernetes/manifests/` is started as a second static Pod.
- **Traps:**
  - `identity` listed first;
  - forgetting the volume mount, so the API server can't find the file and fails to start;
  - forgetting to rewrite existing Secrets;
  - a key that isn't 32 bytes.

## References

- [Encrypting confidential data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) — "Write an encryption configuration file" (the YAML), "Use the new encryption configuration file" (the flag and volume mounts), "Verify that newly written data is encrypted", "Ensure all relevant data are encrypted", "Rotate a decryption key"
- [Using a KMS provider for data encryption](https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/) — KMS v2 setup
- [Secrets Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/) — the alternative: keep Secrets in an external store (Vault, cloud secret managers) and mount them as a CSI volume, so they never sit in etcd

## Related notes

- CKAD: [Secrets](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/05-secrets.md) — the developer's view, with a shorter encryption-at-rest section
