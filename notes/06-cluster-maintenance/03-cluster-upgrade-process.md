# 03 — Cluster Upgrade Process

Upgrading a cluster means moving every component up by one minor version, **control plane first, then the nodes**. The components don't all have to move at once: the **version skew policy** says how far apart they may be, and that gap is what allows a live upgrade.

---

## 1. Version skew: who may lag behind whom

The kube-apiserver is the reference, because every other component talks to it. **Nothing may be newer than the API server.**

| Component | Allowed versions, if the API server is `1.37` |
|---|---|
| kube-apiserver (other HA instances) | Within 1 minor of each other: `1.36`–`1.37` during an upgrade |
| kube-controller-manager, kube-scheduler | Up to **1** minor older: `1.36`–`1.37` |
| kubelet, kube-proxy | Up to **3** minors older: `1.34`–`1.37` |
| kubectl | Within 1 minor, **either direction**: `1.36`–`1.38` |

The course uses `1.10` with kubelets at X-3. At that time the rule was X-2; it became X-3 in 1.28. kube-proxy also may differ from the kubelet on its node by up to 3 minors.

## 2. How far and when

- **One minor version at a time**: `1.35 → 1.36 → 1.37`, never `1.35 → 1.37`. kubeadm refuses to skip.
- Only the 3 newest minors get patches ([02](02-kubernetes-releases.md)), so upgrade before yours drops out.
- Within a minor, patch upgrades (`1.37.1 → 1.37.3`) follow the same steps.

## 3. What is down while the control plane upgrades

With a **single** control plane node, its components restart during the upgrade:

| Still works | Stops until it's back |
|---|---|
| Running Pods keep serving traffic | `kubectl` and every API call |
| The kubelet restarts a crashed **container** in place (its `restartPolicy`) | Deployments, scaling, any change |
| Services keep routing (kube-proxy rules stay) | Controllers: a **Pod** that fails or a node that dies isn't replaced |

With **several** control plane nodes (HA), upgrade them **one at a time**. The load balancer in front of the API servers sends requests to the others, so the API stays up. A 3-member etcd keeps quorum with one member restarting. The skew rule above (API servers within 1 minor of each other) is what makes a half-upgraded control plane legal.

## 4. Strategies for the worker nodes

| Strategy | How | Downtime |
|---|---|---|
| All at once | Upgrade every worker together | **Yes**: no node runs Pods |
| One (or a few) at a time | Drain, upgrade, uncordon, next node | No, if the remaining nodes have capacity |
| Replace nodes | Add new nodes at the new version, drain and remove the old ones | No; easiest in a cloud |

Managed services (GKE, EKS, AKS) do this with a button or one command. On a kubeadm cluster, and on the exam, you do it by hand.

## 5. The kubeadm upgrade

![Three stacked steps. First control plane node: upgrade kubeadm, kubeadm upgrade plan, kubeadm upgrade apply v1.37.x, then drain, upgrade kubelet and kubectl, restart kubelet, uncordon. Other control plane nodes one at a time: upgrade kubeadm, kubeadm upgrade node, then drain, kubelet and kubectl, restart, uncordon. Worker nodes one or a few at a time: upgrade kubeadm, kubeadm upgrade node, drain from a control plane node, upgrade kubelet, restart, uncordon](./diagrams/02-kubeadm-upgrade-order.svg)

**kubeadm doesn't upgrade the kubelet.** `kubeadm upgrade apply` upgrades the control plane static Pods, etcd (when it's stacked), CoreDNS and kube-proxy, and renews the certificates. The kubelet and kubectl are OS packages you upgrade yourself on every node.

### Step 0, on every node: point apt at the new minor

Since 2023 the packages come from `pkgs.k8s.io`, with **one repository per minor version**. Until you change it, apt only sees `1.36` patches.

```bash
sudo sed -i 's|/v1.36/|/v1.37/|' /etc/apt/sources.list.d/kubernetes.list
#   deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.37/deb/ /
sudo apt-get update
apt-cache madison kubeadm        # available versions, e.g. 1.37.3-1.1
```

### First control plane node

```bash
sudo apt-mark unhold kubeadm && sudo apt-get install -y kubeadm='1.37.3-*' && sudo apt-mark hold kubeadm
kubeadm version -o short                    # v1.37.3
sudo kubeadm upgrade plan                   # checks health, shows what can be upgraded and the command
sudo kubeadm upgrade apply v1.37.3          # ... SUCCESS! Your cluster was upgraded to "v1.37.3"

k drain controlplane --ignore-daemonsets
sudo apt-mark unhold kubelet kubectl && \
  sudo apt-get install -y kubelet='1.37.3-*' kubectl='1.37.3-*' && \
  sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload && sudo systemctl restart kubelet
k uncordon controlplane
k get nodes                                 # controlplane now v1.37.3, workers still v1.36.x
```

`apt-mark hold` stops a routine `apt-get upgrade` from moving the Kubernetes packages behind your back.

### Other control plane nodes

The same, with `sudo kubeadm upgrade node` instead of `plan` and `apply`.

### Each worker node

```bash
ssh node01
sudo sed -i 's|/v1.36/|/v1.37/|' /etc/apt/sources.list.d/kubernetes.list && sudo apt-get update
sudo apt-mark unhold kubeadm && sudo apt-get install -y kubeadm='1.37.3-*' && sudo apt-mark hold kubeadm
sudo kubeadm upgrade node                   # on a worker: updates the kubelet configuration

# from the control plane (workers usually have no kubeconfig):
k drain node01 --ignore-daemonsets

# back on node01:
sudo apt-mark unhold kubelet kubectl && \
  sudo apt-get install -y kubelet='1.37.3-*' kubectl='1.37.3-*' && \
  sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload && sudo systemctl restart kubelet

# from the control plane:
k uncordon node01
k get nodes                                 # node01 v1.37.3
```

**Corrections to the slides**, which date from 1.11 / 1.12:

| Slide | Today |
|---|---|
| `apt-get upgrade -y kubeadm=1.12.0-00` | `apt-get install -y kubeadm='1.37.3-*'` after switching the repo; versions now end in `-1.1`, not `-00` |
| `kubeadm upgrade node config --kubelet-version v1.12.0` | `kubeadm upgrade node` (the `config` subcommand was removed) |
| `kubectl drain node-1` | Fails on DaemonSet Pods without `--ignore-daemonsets` ([01](01-os-upgrades-drain-and-cordon.md)) |
| Restart kubelet | Run `systemctl daemon-reload` first: the package replaces the unit files |

### If it fails

`kubeadm upgrade apply` rolls back the control plane if new components don't come up, and is safe to run again. kubeadm keeps backups of the etcd data and the static Pod manifests in `/etc/kubernetes/tmp/`.

---

## Exam angle

- **"Upgrade the control plane and node01 from 1.36.x to 1.37.x":** a classic, long task. Follow the order above; it scores per node, so finish the control plane completely before starting the worker.
- **Use the exact version the task names** (`1.37.3`), not the latest one `upgrade plan` suggests.
- **Where each command runs:** `kubeadm`, `apt` and `systemctl` on the node being upgraded (`ssh node01`, then `sudo`); `kubectl drain/uncordon` from the control plane. Remember to `exit` back.
- **Verify** with `k get nodes` (VERSION is the kubelet, so it only changes after the kubelet restart) and `kubeadm upgrade plan` / `k version` for the control plane.
- **Traps:** forgetting to switch the apt repo (apt finds no 1.37 package); skipping `kubeadm upgrade node` on a worker; forgetting `uncordon`.

## References

- [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/) — "Upgrading control plane nodes" (the full command blocks), "Recovering from a failure state", "How it works"
- [Upgrading Linux nodes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/) — "Upgrading worker nodes", step by step
- [Changing the Kubernetes package repository](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/change-package-repository/) — the `kubernetes.list` line to edit
- [Version skew policy](https://kubernetes.io/releases/version-skew-policy/) — "Supported version skew", "Supported component upgrade order"
