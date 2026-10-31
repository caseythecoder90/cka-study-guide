# etcd

`etcdctl` and `etcdutl`, from a standalone lab install to the cluster's etcd. Chapters: [02-03 etcd for Beginners](../02-core-concepts/03-etcd-for-beginners.md).

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

## etcdctl vs etcdutl

| Task | Tool |
|---|---|
| Anything against a running server (get, put, watch, member list, **snapshot save**) | `etcdctl` |
| Work on a data directory or snapshot file offline (**snapshot restore**, snapshot status, defrag) | `etcdutl` — `etcdctl snapshot restore` was removed in etcd v3.6 |

Against a Kubernetes cluster's etcd, every `etcdctl` command also needs `--endpoints`, `--cacert`, `--cert` and `--key`; see [`../exam-facts.md`](../exam-facts.md#etcd-backup-and-restore).
