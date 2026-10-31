# Exam facts — recall sheet

> Everything you would otherwise have to hunt for during the exam: paths on a node, ports, flags, certificate locations, version-skew rules. Grouped by topic, not by course section, because tasks are. Updated in the same PR as every chapter.
>
> Values below are kubeadm defaults. A task cluster can differ — when in doubt, read the static Pod manifest or `ps aux | grep <component>` on the node rather than trusting this sheet.

## The exam itself

| Fact | Value |
|---|---|
| Format | Performance-based: tasks on live clusters, in a PSI remote desktop |
| Tasks / time | 15-20 tasks in 2 hours |
| Passing score | 66% |
| Validity | 2 years |
| Included | One free retake, two killer.sh simulator sessions |
| Allowed resources | kubernetes.io docs and blog (plus any others the candidate handbook lists). Confirm the current list before the exam |
| Domains and weights | Troubleshooting 30% · Cluster Architecture, Installation & Configuration 25% · Services & Networking 20% · Workloads & Scheduling 15% · Storage 10% |
| First thing in every task | Run the `kubectl config use-context ...` line the task gives you. Wrong cluster = zero points |

## Paths on a control-plane node (kubeadm)

| What | Path |
|---|---|
| Static Pod manifests (API server, etcd, scheduler, controller manager) | `/etc/kubernetes/manifests/` |
| Cluster PKI (CA, API server, front proxy, SA keys) | `/etc/kubernetes/pki/` |
| etcd PKI | `/etc/kubernetes/pki/etcd/` |
| Admin kubeconfig | `/etc/kubernetes/admin.conf` |
| Component kubeconfigs | `/etc/kubernetes/{kubelet,controller-manager,scheduler,super-admin}.conf` |
| kubelet config (includes `staticPodPath`) | `/var/lib/kubelet/config.yaml` |
| kubelet systemd drop-in | `/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf` (path varies by distro: also check `/etc/systemd/system/kubelet.service.d/`) |
| etcd data directory | `/var/lib/etcd` |
| CNI config | `/etc/cni/net.d/` |
| CNI plugin binaries | `/opt/cni/bin/` |
| User kubeconfig | `~/.kube/config` (or `$KUBECONFIG`) |

## Ports

| Component | Port |
|---|---|
| kube-apiserver | 6443 |
| etcd client / peer | 2379 / 2380 |
| kubelet API | 10250 |
| kube-scheduler | 10259 |
| kube-controller-manager | 10257 |
| kube-proxy health | 10256 |
| kube-proxy metrics (`/proxyMode` shows the mode) | 10249 |
| NodePort range | 30000-32767 |

## How each component runs (kubeadm)

| Component | Runs as | Fix it with | Chapter |
|---|---|---|---|
| kube-apiserver, etcd, kube-scheduler, kube-controller-manager | Static Pods on control plane nodes | Edit `/etc/kubernetes/manifests/<component>.yaml`; kubelet recreates the Pod | [02-01](02-core-concepts/01-cluster-architecture.md) |
| kubelet | systemd service on every node — **not installed by kubeadm** | `systemctl`, `journalctl -u kubelet`, `/var/lib/kubelet/config.yaml` | [02-01](02-core-concepts/01-cluster-architecture.md) |
| containerd | systemd service on every node | `systemctl status containerd`, `crictl` | [02-01](02-core-concepts/01-cluster-architecture.md) |
| kube-proxy | DaemonSet in `kube-system` | `kubectl -n kube-system logs ds/kube-proxy`, ConfigMap `kube-proxy` | [02-01](02-core-concepts/01-cluster-architecture.md) |
| CoreDNS | Deployment in `kube-system` (Service `kube-dns`) | `kubectl -n kube-system ...` | [02-01](02-core-concepts/01-cluster-architecture.md) |

## Architecture rules

| Fact | Value | Chapter |
|---|---|---|
| Only component that talks to etcd | kube-apiserver | [02-01](02-core-concepts/01-cluster-architecture.md) |
| API server request pipeline | Authentication → authorization → admission (mutating, then validating) → schema validation → etcd | [02-01](02-core-concepts/01-cluster-architecture.md) |
| API server calls the kubelet (:10250) for | `logs`, `exec`, `attach`, `port-forward` | [02-01](02-core-concepts/01-cluster-architecture.md) |
| Node heartbeat | `Lease` per node in namespace `kube-node-lease`, renewed about every 10 s | [02-01](02-core-concepts/01-cluster-architecture.md) |
| Runtime chain | kubelet → CRI (gRPC) → containerd → containerd-shim-runc-v2 (one per Pod) → runc → kernel | [02-02](02-core-concepts/02-docker-vs-containerd.md) |
| runc | Low-level OCI runtime: creates namespaces + cgroups, starts the process, **then exits** | [02-02](02-core-concepts/02-docker-vs-containerd.md) |
| CRI sockets | containerd `/run/containerd/containerd.sock` · CRI-O `/var/run/crio/crio.sock` · cri-dockerd `/var/run/cri-dockerd.sock` | [02-02](02-core-concepts/02-docker-vs-containerd.md) |
| crictl config | `/etc/crictl.yaml` (`runtime-endpoint`), or `--runtime-endpoint`, or `$CONTAINER_RUNTIME_ENDPOINT` | [02-02](02-core-concepts/02-docker-vs-containerd.md) |
| kubelet's runtime socket | `containerRuntimeEndpoint` in `/var/lib/kubelet/config.yaml` | [02-02](02-core-concepts/02-docker-vs-containerd.md) |
| containerd namespace used by Kubernetes | `k8s.io` (`ctr -n k8s.io ...`); Docker uses `moby`; `ctr` defaults to `default` | [02-02](02-core-concepts/02-docker-vs-containerd.md) |
| CRI introduced | v1.5 (Dec 2016) | [02-02](02-core-concepts/02-docker-vs-containerd.md) |
| dockershim | Deprecated v1.20, **removed v1.24**. Docker-built images still run — they are OCI images | [02-02](02-core-concepts/02-docker-vs-containerd.md) |
| containerd | Donated to the CNCF by Docker in 2017, graduated 2019 | [02-02](02-core-concepts/02-docker-vs-containerd.md) |
| OCI specs | image-spec, runtime-spec (plus distribution-spec for registries) | [02-02](02-core-concepts/02-docker-vs-containerd.md) |
| CLIs | `ctr` (containerd, debugging) · `nerdctl` (containerd, Docker-like, general use) · `crictl` (Kubernetes, any CRI runtime, Pod-aware, debugging) | [02-02](02-core-concepts/02-docker-vs-containerd.md) |
| kube-proxy default mode on Linux | `iptables` (others: `nftables`, `ipvs`) | [02-01](02-core-concepts/01-cluster-architecture.md) |
| kube-proxy iptables chains | `KUBE-SERVICES` → `KUBE-SVC-<hash>` (random pick) → `KUBE-SEP-<hash>` (DNAT); NodePorts in `KUBE-NODEPORTS` | [02-01](02-core-concepts/01-cluster-architecture.md) |
| Who routes Pod-to-Pod traffic | The CNI plugin, not kube-proxy | [02-01](02-core-concepts/01-cluster-architecture.md) |

## Cluster defaults (kubeadm)

| Fact | Value |
|---|---|
| Service CIDR | `10.96.0.0/12` |
| Cluster DNS Service IP | `10.96.0.10` (the `kube-dns` Service, backed by CoreDNS) |
| Cluster domain | `cluster.local` |
| Leaf certificate validity | 1 year. `kubeadm upgrade` renews them; `kubeadm certs check-expiration` shows them |
| CA certificate validity | 10 years |

## Version skew and upgrades

| Rule | Value |
|---|---|
| Upgrade step | One minor version at a time (1.33 → 1.34, never 1.33 → 1.35) |
| Order | Control plane first (`kubeadm upgrade apply` on the first, `kubeadm upgrade node` on the rest), then each worker: drain → upgrade kubeadm → `kubeadm upgrade node` → upgrade kubelet + kubectl → restart kubelet → uncordon |
| kubelet vs API server | kubelet may be up to 3 minor versions **older**, never newer |
| Controller manager / scheduler vs API server | Up to 1 minor older, never newer |
| kubectl vs API server | Within 1 minor, either direction |

## etcd

| Fact | Value | Chapter |
|---|---|---|
| What it is | Distributed, reliable key-value store; Raft consensus; written in Go | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| Ports | 2379 client · 2380 peer (member to member) | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| Quorum | `floor(n/2)+1` — 3 members tolerate 1 failure, 5 tolerate 2; use odd counts | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| Writes | Only the leader accepts them; committed once a quorum stores them; no quorum = no writes | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| Binaries | `etcd` server · `etcdctl` client (needs a running server) · `etcdutl` offline tool (data dir and snapshot files) | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| v2 vs v3 commands | `set`/`get`/`rm` vs `put`/`get`/`del`/`txn`; version: `etcdctl --version` vs `etcdctl version` | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| `ETCDCTL_API` default | v2 before etcd v3.4, **v3 from v3.4**. Kubernetes uses only the v3 API | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| `etcdctl snapshot restore` | Deprecated v3.5, **removed v3.6** → `etcdutl snapshot restore` | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| etcd version kubeadm deploys | 3.7 for Kubernetes v1.35 and later | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| `resourceVersion` | Comes from the etcd revision of the object's last write; drives `409 Conflict` | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| API server compaction of old revisions | Every 5 min (`--etcd-compaction-interval`) | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| History | Created at CoreOS 2013 · v3 API in v3.0 (Jun 2016) · CNCF incubating Dec 2018 · graduated Nov 2020 | [02-03](02-core-concepts/03-etcd-for-beginners.md) |
| How etcd runs | kubeadm: static Pod `etcd-<node>` in `kube-system`, manifest `/etc/kubernetes/manifests/etcd.yaml`. From scratch: systemd `etcd.service` | [02-04](02-core-concepts/04-etcd-in-kubernetes.md) |
| etcd flags to read from the manifest | `--data-dir`, `--listen-client-urls`, `--advertise-client-urls`, `--cert-file`, `--key-file`, `--trusted-ca-file`, `--initial-cluster` | [02-04](02-core-concepts/04-etcd-in-kubernetes.md) |
| Listen vs advertise | listen = addresses etcd binds to; advertise = address published for others to use | [02-04](02-core-concepts/04-etcd-in-kubernetes.md) |
| API server → etcd flags | `--etcd-servers`, `--etcd-cafile` (`pki/etcd/ca.crt`), `--etcd-certfile`/`--etcd-keyfile` (`pki/apiserver-etcd-client.crt/.key`) | [02-04](02-core-concepts/04-etcd-in-kubernetes.md) |
| Key layout | `/registry/<resource>/<namespace>/<name>`; cluster-scoped `/registry/<resource>/<name>`; nodes `/registry/minions/<name>`; non-core groups `/registry/<group>/<resource>/...` | [02-04](02-core-concepts/04-etcd-in-kubernetes.md) |
| Value encoding | protobuf. Secrets are unencrypted in etcd unless encryption at rest is configured | [02-04](02-core-concepts/04-etcd-in-kubernetes.md) |
| HA topologies | Stacked (kubeadm default; one member per control plane node, API server uses its local member) vs external (dedicated etcd hosts). One member per machine | [02-04](02-core-concepts/04-etcd-in-kubernetes.md) |

## etcd backup and restore

```bash
ETCDCTL_API=3 etcdctl snapshot save /path/snap.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

The cert paths are in `/etc/kubernetes/manifests/etcd.yaml` (`--trusted-ca-file`, `--cert-file`, `--key-file`) — read them from there instead of memorizing. `snapshot save` stays in `etcdctl` because it needs the running server. **Restore is `etcdutl snapshot restore`** on etcd v3.6 and later (what current kubeadm deploys); full restore steps are added when Cluster Maintenance is written.
