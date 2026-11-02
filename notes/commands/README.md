# Commands — per-topic reference

This directory splits the global [`../commands.md`](../commands.md) into focused files, one per task area. Use these when drilling a single topic; use `../commands.md` when you want one page to `Ctrl+F`.

Both stay in sync — when a chapter adds new commands, update **both** the relevant file here and the global reference.

## Index

| File | What's in it | Chapter |
|---|---|---|
| [`setup.md`](setup.md) | Aliases (`k`, `$do`, `$now`), completion, vim config and essentials, switching context before every task, getting onto a node | — |
| [`imperative.md`](imperative.md) | **The exam time-saver.** Imperative generators with `--dry-run=client -o yaml`: workloads, access control (RBAC, ServiceAccounts, tokens), quotas, PriorityClasses, Ingress, and which resources have no generator | — |
| [`cluster-architecture.md`](cluster-architecture.md) | Finding and checking each component: control plane static Pods and `crictl` when the API server is down, kubelet and containerd with `systemctl`/`journalctl`, node heartbeats, `kubectl -v` to see the REST calls, and reading kube-proxy's iptables chains for one Service | [02-01](../02-core-concepts/01-cluster-architecture.md) |
| [`container-runtimes.md`](container-runtimes.md) | `crictl` for Pods, containers, logs, exec and images; pointing it at the CRI socket with `/etc/crictl.yaml`; `ctr` and containerd namespaces (`k8s.io`); `nerdctl` as a Docker replacement; checking the containerd daemon and its shims | [02-02](../02-core-concepts/02-docker-vs-containerd.md) |
| [`etcd.md`](etcd.md) | Installing etcd from release binaries, v2 vs v3 `etcdctl` and `ETCDCTL_API`, put/get/del with prefixes, watch and revisions, endpoint and member health; the cluster's etcd on kubeadm (exec into the Pod with certificates, `ETCDCTL_*` variables, `/registry` keys) and from scratch (systemd); `etcdctl` versus `etcdutl` | [02-03](../02-core-concepts/03-etcd-for-beginners.md) · [02-04](../02-core-concepts/04-etcd-in-kubernetes.md) |
| [`kube-apiserver.md`](kube-apiserver.md) | Seeing the REST calls behind kubectl (`-v`, `get --raw`), finding the API server's options in the kubeadm manifest, the systemd unit or `ps`, recovering it when kubectl gets connection refused (`crictl`, backup outside `manifests/`), downloading the binary from dl.k8s.io | [02-05](../02-core-concepts/05-kube-apiserver.md) |
| [`kube-controller-manager.md`](kube-controller-manager.md) | Finding its options (manifest, unit file, `ps`), checking it works with a test Deployment, the leader-election Lease, fixing the static Pod manifest, and watching the node controller taint a failed node and evict its Pods after the 300 s toleration | [02-06](../02-core-concepts/06-kube-controller-manager.md) |
| [`kube-scheduler.md`](kube-scheduler.md) | Finding its options (manifest, unit file, `--config` file, `ps`), the leader Lease, reading its decisions (NODE column, unscheduled Pods, `FailedScheduling` events, allocated requests on a node), and fixing it when Pods stay Pending | [02-07](../02-core-concepts/07-kube-scheduler.md) |
| [`kubelet.md`](kubelet.md) | Checking the kubelet on a node (`systemctl`, `journalctl`), reading its settings (process flags, unit and drop-in, `kubeadm-flags.env`, `config.yaml`, `kubelet.conf`), fixing and restarting it, what it is running, and its live config via `configz` | [02-08](../02-core-concepts/08-kubelet.md) |
| [`kube-proxy.md`](kube-proxy.md) | Checking the DaemonSet covers every node, reading and changing its ConfigMap and restarting it, the from-scratch systemd service, and on a node: its mode and health endpoints, a Service's endpoints and iptables rules, and a test request from a Pod | [02-09](../02-core-concepts/09-kube-proxy.md) |
