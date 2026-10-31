# etcd

`etcdctl` and `etcdutl`, from a standalone lab install to the cluster's etcd. Chapters: [02-03 etcd for Beginners](../02-core-concepts/03-etcd-for-beginners.md) · [02-04 etcd in Kubernetes](../02-core-concepts/04-etcd-in-kubernetes.md).

## Standalone lab install

```bash
ETCD_VER=v3.x.y                                  # a current release from github.com/etcd-io/etcd/releases
curl -L https://github.com/etcd-io/etcd/releases/download/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz \
  -o etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf etcd-${ETCD_VER}-linux-amd64.tar.gz
cd etcd-${ETCD_VER}-linux-amd64
./etcd                                           # single member on localhost:2379 / 2380
```

## Which API

```bash
etcdctl version                                  # v3 syntax; shows etcdctl and API version
etcdctl --version                                # v2 syntax
export ETCDCTL_API=3                             # needed only on etcdctl older than v3.4
```

| Operation | v2 | v3 |
|---|---|---|
| Write | `set` | `put` |
| Read | `get` | `get` |
| Delete | `rm` | `del` |
| Transaction | — | `txn` |

## Keys and values (v3)

```bash
etcdctl put key1 value1
etcdctl get key1
etcdctl get key1 --print-value-only
etcdctl get key1 -w json                         # create_revision, mod_revision, version
etcdctl put /app/config/mode prod
etcdctl get /app/config --prefix
etcdctl get "" --prefix --keys-only              # list every key
etcdctl del key1
etcdctl del /app --prefix
```

## Watch and revisions

```bash
etcdctl watch key1                               # blocks; prints each PUT/DELETE
etcdctl watch /app --prefix
etcdctl watch key1 --rev=1                       # replay history (until compacted)
```

## Cluster health

```bash
etcdctl endpoint health
etcdctl endpoint status -w table                 # leader, DB size, raft term, revision
etcdctl member list -w table
```

## The cluster's etcd (kubeadm)

```bash
kubectl -n kube-system get pods | grep etcd                # etcd-<node-name>
cat /etc/kubernetes/manifests/etcd.yaml                    # flags, cert paths, data-dir hostPath
grep etcd /etc/kubernetes/manifests/kube-apiserver.yaml    # how the API server reaches it

# From the workstation, through the Pod (pass flags directly; the image may have no shell)
kubectl -n kube-system exec etcd-controlplane -- etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get / --prefix --keys-only --limit=10

# Or on the control plane node itself, with etcdctl installed there
export ETCDCTL_ENDPOINTS=https://127.0.0.1:2379
export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
etcdctl get /registry/pods --prefix --keys-only            # every Pod
etcdctl get /registry/minions --prefix --keys-only         # nodes
etcdctl get /registry/secrets/default --prefix --keys-only
etcdctl member list -w table
etcdctl endpoint status -w table
```

`ETCDCTL_<FLAG>` environment variables stand in for the global flags (upper-case, dashes to underscores), which saves retyping the certificates for every command.

## The cluster's etcd (from scratch)

```bash
systemctl status etcd
cat /etc/systemd/system/etcd.service                       # flags in ExecStart
journalctl -u etcd --no-pager | tail -30
systemctl daemon-reload && systemctl restart etcd          # after editing the unit
```

## etcdctl vs etcdutl

| Task | Tool |
|---|---|
| Anything against a running server (get, put, watch, member list, **snapshot save**) | `etcdctl` |
| Work on a data directory or snapshot file offline (**snapshot restore**, snapshot status, defrag) | `etcdutl` — `etcdctl snapshot restore` was removed in etcd v3.6 |

Against a Kubernetes cluster's etcd, every `etcdctl` command also needs `--endpoints`, `--cacert`, `--cert` and `--key`; see [`../exam-facts.md`](../exam-facts.md#etcd-backup-and-restore).
