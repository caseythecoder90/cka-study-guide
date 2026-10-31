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
