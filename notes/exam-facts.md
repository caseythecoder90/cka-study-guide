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
| API server request pipeline | Authentication (401) → authorization (403) → mutating admission → schema validation (422) → validating admission → etcd | [02-05](02-core-concepts/05-kube-apiserver.md) |
| How components "monitor" the API server | List, then a long-lived **watch** over HTTPS; the scheduler watches Pods with empty `spec.nodeName` and POSTs to `pods/<name>/binding` | [02-05](02-core-concepts/05-kube-apiserver.md) |
| Component kubeconfigs (kubeadm) | `/etc/kubernetes/{scheduler,controller-manager,kubelet,admin,super-admin}.conf` | [02-05](02-core-concepts/05-kube-apiserver.md) |
| Where kube-apiserver's options are | kubeadm `/etc/kubernetes/manifests/kube-apiserver.yaml` · from scratch `/etc/systemd/system/kube-apiserver.service` · either `ps -aux \| grep kube-apiserver` | [02-05](02-core-concepts/05-kube-apiserver.md) |
| kube-apiserver flags kubeadm sets | `--authorization-mode=Node,RBAC`, `--enable-admission-plugins=NodeRestriction`, `--service-cluster-ip-range=10.96.0.0/12`, `--etcd-servers=https://127.0.0.1:2379`, `--secure-port=6443` | [02-05](02-core-concepts/05-kube-apiserver.md) |
| kube-apiserver certificates (kubeadm, under `/etc/kubernetes/pki/`) | serving `apiserver.crt` · client CA `ca.crt` · to etcd `apiserver-etcd-client.*` · to kubelets `apiserver-kubelet-client.*` · to aggregated APIs `front-proxy-client.*` · SA tokens `sa.key`/`sa.pub` | [02-05](02-core-concepts/05-kube-apiserver.md) |
| Removed flags still on old slides | `--insecure-port`, `--experimental-encryption-provider-config` (→ `--encryption-provider-config`), `--enable-swagger-ui`, `--kubelet-https`, `--apiserver-count`, admission plugin `Initializers` | [02-05](02-core-concepts/05-kube-apiserver.md) |
| Kubernetes binaries download | `https://dl.k8s.io/release/<version>/bin/linux/amd64/<binary>` | [02-05](02-core-concepts/05-kube-apiserver.md) |
| Controller | Loop: watch → compare desired vs actual → act, through the API server | [02-06](02-core-concepts/06-kube-controller-manager.md) |
| Node controller timings | `--node-monitor-period=5s` · `--node-monitor-grace-period=50s` (was 40s) → `Ready=Unknown` + taint `node.kubernetes.io/unreachable` | [02-06](02-core-concepts/06-kube-controller-manager.md) |
| Pod eviction from a failed node | Taint-based: default toleration `tolerationSeconds: 300` added by `DefaultTolerationSeconds`. **`--pod-eviction-timeout` removed** | [02-06](02-core-concepts/06-kube-controller-manager.md) |
| kube-controller-manager | One binary, ~45 controllers; port 10257; `--controllers=*` (all defaults), `foo` enable, `-foo` disable | [02-06](02-core-concepts/06-kube-controller-manager.md) |
| Disabled-by-default controllers | `bootstrap-signer-controller`, `token-cleaner-controller`, `selinux-warning-controller`. kubeadm: `--controllers=*,bootstrapsigner,tokencleaner` | [02-06](02-core-concepts/06-kube-controller-manager.md) |
| Leader election | `--leader-elect=true` (default): one active controller manager (and scheduler); lock is a `Lease` in `kube-system` | [02-06](02-core-concepts/06-kube-controller-manager.md) |
| kube-controller-manager flags kubeadm sets | `--bind-address=127.0.0.1`, `--kubeconfig=/etc/kubernetes/controller-manager.conf`, `--use-service-account-credentials=true`, `--cluster-signing-cert-file/key-file=pki/ca.crt/.key`, `--service-account-private-key-file=pki/sa.key`, `--root-ca-file=pki/ca.crt` | [02-06](02-core-concepts/06-kube-controller-manager.md) |
| Removed controller manager flags on old slides | `--address` (→ `--bind-address`), `--pod-eviction-timeout` | [02-06](02-core-concepts/06-kube-controller-manager.md) |
| Scheduler's job | Decides the node only (writes `spec.nodeName` via a Binding); the kubelet creates the Pod | [02-07](02-core-concepts/07-kube-scheduler.md) |
| Scheduling phases | **Filter** (remove nodes that cannot run it) → **Score** (rank the rest) → bind. Filtering uses **requests**, not live usage | [02-07](02-core-concepts/07-kube-scheduler.md) |
| Scores | Each scoring plugin returns **0-100**, summed by weight (old slides: 0-10). Ties broken at random. Default resource strategy `LeastAllocated` (most free wins) | [02-07](02-core-concepts/07-kube-scheduler.md) |
| No node fits | Pod stays `Pending`, event `FailedScheduling: 0/N nodes are available: ...` naming each failing filter | [02-07](02-core-concepts/07-kube-scheduler.md) |
| kubelet's jobs | Register the node · create Pods bound to it (and static Pods) via CRI · monitor and report status + heartbeat Lease | [02-08](02-core-concepts/08-kubelet.md) |
| kubelet install | **Not deployed by kubeadm** — `kubelet` package on every node (control plane too), systemd service, binary `/usr/bin/kubelet` | [02-08](02-core-concepts/08-kubelet.md) |
| kubelet files (kubeadm) | drop-in `/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf` · `/var/lib/kubelet/config.yaml` · `/etc/kubernetes/kubelet.conf` · `/etc/kubernetes/bootstrap-kubelet.conf` · `/var/lib/kubelet/kubeadm-flags.env` · `/etc/default/kubelet` (`/etc/sysconfig/kubelet`) | [02-08](02-core-concepts/08-kubelet.md) |
| kubelet cgroup driver | `cgroupDriver` in `config.yaml`; kubeadm defaults to `systemd`; must match containerd | [02-08](02-core-concepts/08-kubelet.md) |
| Removed kubelet flags on old slides | `--container-runtime`, `--image-pull-progress-deadline`, `--network-plugin`, `--cni-bin-dir`, `--cni-conf-dir` (dockershim era). `--cgroup-driver` and `--container-runtime-endpoint` belong in `config.yaml` | [02-08](02-core-concepts/08-kubelet.md) |
| kube-scheduler | Port 10259; kubeadm flags `--kubeconfig=/etc/kubernetes/scheduler.conf`, `--bind-address=127.0.0.1`, `--leader-elect=true`; settings file via `--config` (`KubeSchedulerConfiguration`). `--address` removed | [02-07](02-core-concepts/07-kube-scheduler.md) |
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
| Service IP | Not on the Pod network; no interface, no process. Allocated from `--service-cluster-ip-range` (kubeadm `10.96.0.0/12`); exists only as rules kube-proxy writes on every node | [02-09](02-core-concepts/09-kube-proxy.md) |
| kube-proxy (kubeadm) | DaemonSet `kube-proxy` in `kube-system`; ConfigMap `kube-proxy` keys `config.conf` + `kubeconfig.conf` at `/var/lib/kube-proxy`; nodeSelector `kubernetes.io/os=linux`; tolerates all taints; `hostNetwork`, privileged | [02-09](02-core-concepts/09-kube-proxy.md) |
| Pod | Smallest object you can create; a single instance of an application. Scale = more Pods, never more copies of a container in one Pod | [02-10](02-core-concepts/10-pods.md) |
| Containers in one Pod share | Lifecycle, node, network namespace (one IP, `localhost`, distinct ports), volumes | [02-10](02-core-concepts/10-pods.md) |
| Native sidecar | `initContainers` entry with `restartPolicy: Always`; stable since v1.33 | [02-10](02-core-concepts/10-pods.md) |
| `kubectl run` | Creates only a Pod, named exactly as given, since v1.18 (older versions created a Deployment) | [02-10](02-core-concepts/10-pods.md) |
| Image with no registry | `nginx` = `docker.io/library/nginx:latest` | [02-10](02-core-concepts/10-pods.md) |
| Manifest fields | `apiVersion`, `kind`, `metadata` on every object; `spec` on most. ConfigMap/Secret use `data`, Role `rules`, RoleBinding `subjects` + `roleRef` | [02-11](02-core-concepts/11-pods-with-yaml.md) |
| YAML indentation | Spaces only, tabs are a parse error; siblings match; children deeper; count does not matter (2 by convention) | [02-11](02-core-concepts/11-pods-with-yaml.md) |
| Must be quoted in label and env values | Numbers and booleans: `"1"`, `"yes"` | [02-11](02-core-concepts/11-pods-with-yaml.md) |
| API versions | `v1` Pod, Service, ConfigMap, Secret, Namespace, SA, PV, PVC · `apps/v1` Deployment, ReplicaSet, DaemonSet, StatefulSet · `batch/v1` Job, CronJob · `networking.k8s.io/v1` Ingress, NetworkPolicy · `rbac.authorization.k8s.io/v1` RBAC · `storage.k8s.io/v1` StorageClass. Check: `kubectl api-resources` | [02-11](02-core-concepts/11-pods-with-yaml.md) |
| `create` vs `apply` vs `replace` | `create -f` errors if it exists · `apply -f` creates or updates · `replace --force -f` deletes and recreates | [02-11](02-core-concepts/11-pods-with-yaml.md) |
| ReplicaSet vs ReplicationController | RS: `apps/v1`, `spec.selector` required, `matchLabels` and `matchExpressions`. RC: `v1`, selector optional, equality only, legacy | [02-12](02-core-concepts/12-replicasets.md) |
| ReplicaSet selector | Must match `spec.template.metadata.labels` or the API rejects it. Adopts any matching Pods in its namespace | [02-12](02-core-concepts/12-replicasets.md) |
| Template edits on a ReplicaSet | Do not change running Pods; delete them to recreate from the new template | [02-12](02-core-concepts/12-replicasets.md) |
| Scaling | `kubectl scale --replicas=N rs/<name>` or `-f file`; neither updates the file, and a later `apply` reverts it | [02-12](02-core-concepts/12-replicasets.md) |
| Delete without Pods | `kubectl delete rs <name> --cascade=orphan` | [02-12](02-core-concepts/12-replicasets.md) |
| ReplicaSet generator | None. Generate a Deployment, change `kind`, delete `strategy` | [02-12](02-core-concepts/12-replicasets.md) |
| Deployment chain | Deployment → one ReplicaSet per template version → Pods. Old ReplicaSets stay at 0 replicas for rollback | [02-13](02-core-concepts/13-deployments.md) |
| Rollout trigger | Only a change to `spec.template`. Scaling is not a rollout and adds no revision | [02-13](02-core-concepts/13-deployments.md) |
| Strategies | `RollingUpdate` (default) or `Recreate`, which kills all old Pods first. `Recreate` must not have a `rollingUpdate` block | [02-13](02-core-concepts/13-deployments.md) |
| `maxSurge` | Pods allowed above `replicas`; number or %; % rounds **up**; default 25% | [02-13](02-core-concepts/13-deployments.md) |
| `maxUnavailable` | Pods allowed unavailable below `replicas`; number or %; % rounds **down**; default 25%. Cannot both be 0 | [02-13](02-core-concepts/13-deployments.md) |
| `minAvailable` | A PodDisruptionBudget field, not a Deployment field. PDBs do not limit rolling updates | [02-13](02-core-concepts/13-deployments.md) |
| Deployment defaults | `minReadySeconds` 0 · `progressDeadlineSeconds` 600 · `revisionHistoryLimit` 10 · selector immutable | [02-13](02-core-concepts/13-deployments.md) |
| `kubectl get all` | Skips ConfigMaps, Secrets, Ingresses, PVCs and others | [02-13](02-core-concepts/13-deployments.md) |
| Service types | ClusterIP (default, internal) · NodePort (+ same port on every node) · LoadBalancer (+ cloud load balancer) · ExternalName (DNS alias only) | [02-14](02-core-concepts/14-services.md) |
| NodePort fields | `port` required · `targetPort` defaults to `port` · `nodePort` allocated from 30000-32767 (`--service-node-port-range`) if omitted | [02-14](02-core-concepts/14-services.md) |
| Reading the output | `PORT(S)` `80:30008/TCP` = `port:nodePort`. `--tcp=80:8080` = `port:targetPort` | [02-14](02-core-concepts/14-services.md) |
| Service selector | Plain `key: value`, no `matchLabels`. Only Ready Pods in the same namespace receive traffic | [02-14](02-core-concepts/14-services.md) |
| EndpointSlice | `discovery.k8s.io/v1`, label `kubernetes.io/service-name=<svc>`, 100 endpoints per slice by default. `Endpoints` deprecated since v1.33 | [02-14](02-core-concepts/14-services.md) |
| Service generators | `kubectl expose` copies the selector, has no `--node-port`. `kubectl create service` sets selector `app=<service-name>`, has `--node-port` | [02-14](02-core-concepts/14-services.md) |
| ExternalName | `spec.externalName` returns a CNAME from cluster DNS. No ClusterIP, selector, endpoints, proxying or port mapping. DNS names only, not IPs. HTTP Host and TLS names can break | [02-14](02-core-concepts/14-services.md) |
| ClusterIP | Default Service type; internal only. IP from `--service-cluster-ip-range` (kubeadm `10.96.0.0/12`). `kubernetes` Service is usually `10.96.0.1` | [02-15](02-core-concepts/15-services-clusterip.md) |
| Service DNS | `svc` same namespace · `svc.ns` another namespace · `svc.ns.svc.cluster.local` fully qualified. No namespace = Pod's own namespace only | [02-15](02-core-concepts/15-services-clusterip.md) |
| CoreDNS Pods | `kubectl -n kube-system get pods -l k8s-app=kube-dns` | [02-15](02-core-concepts/15-services-clusterip.md) |
| Headless Service | `clusterIP: None`: no IP, no kube-proxy; DNS returns Pod IPs | [02-15](02-core-concepts/15-services-clusterip.md) |
| Change kube-proxy settings | Edit ConfigMap `kube-proxy`, then `kubectl -n kube-system rollout restart daemonset kube-proxy` | [02-09](02-core-concepts/09-kube-proxy.md) |
| kube-proxy from scratch | systemd service `kube-proxy --config=<KubeProxyConfiguration file>` | [02-09](02-core-concepts/09-kube-proxy.md) |

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
