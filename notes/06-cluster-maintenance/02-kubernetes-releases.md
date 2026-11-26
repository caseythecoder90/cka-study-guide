# 02 — Kubernetes Releases

A Kubernetes version looks like `v1.33.0`:

| Part | | Changes when | Example |
|---|---|---|---|
| **Major** | `1` | Never so far: every release since July 2015 is 1.x | |
| **Minor** | `33` | About **3 times a year**, with new features | 1.33 → 1.34 |
| **Patch** | `0` | About **monthly**, with bug and security fixes only | 1.33.0 → 1.33.1 |

The project supports the **3 newest minor versions** at a time, each with about **1 year of patches**. In November 2026 that's 1.37, 1.36 and 1.35. Upgrades move one minor version at a time; the next chapters cover how.

---

## 1. "Alpha" and "beta" mean three different things

This is why the topic is confusing: the same two words label three separate things.

| What | Looks like | Means |
|---|---|---|
| **A pre-release of Kubernetes itself** | `v1.33.0-alpha.1`, `-beta.0`, `-rc.0` | Test builds published **before** v1.33.0 comes out. Never run them in production |
| **A feature's maturity** | "In-place resize: alpha in 1.27, beta in 1.33, GA in 1.35" | How far one feature has come. It moves up over **several** releases |
| **An API's version** | `v1alpha1`, `v1beta1`, `v1`, as in `autoscaling/v2` | Which version of a resource's schema you write in `apiVersion` |

The slide with `v1.10.0-alpha` and `v1.10.0-beta` mixes the first two up. Feature defaults belong to the **feature**, not to a pre-release build:

| Feature stage | On by default? | How it's switched |
|---|---|---|
| Alpha | **No** | `--feature-gates=Name=true` on the components |
| Beta | Usually **yes** (but **new beta APIs are off** by default since 1.24) | Feature gate; for an API, `--runtime-config` on the kube-apiserver |
| GA (stable) | **Always**; can't be turned off | Nothing; the gate is later removed |

The course has already shown each stage: in-place resize was alpha in its video and is GA now ([05-12](../05-application-lifecycle-management/12-in-place-pod-resize.md)); native sidecars and `autoscaling/v2` are stable ([05-09](../05-application-lifecycle-management/09-multi-container-design-patterns.md), [05-11](../05-application-lifecycle-management/11-horizontal-pod-autoscaler.md)).

## 2. What shares a version number

| Component | Version |
|---|---|
| kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, kube-proxy, kubectl, kubeadm | **Kubernetes' version**: all from the same release |
| etcd | **Its own** (3.x) |
| CoreDNS | **Its own** (1.x) |
| Container runtime (containerd), CNI plugin | Their own |

Each Kubernetes release lists the etcd and CoreDNS versions it was tested with, and `kubeadm` installs those.

## 3. Finding versions

```bash
k version                         # Client (kubectl) and Server (kube-apiserver)
k get nodes                       # VERSION = each node's kubelet
kubeadm version -o short
kubelet --version                 # on a node
k -n kube-system get pods -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image
                                  # etcd, CoreDNS and control-plane image tags
k api-versions                    # API versions the server offers, e.g. autoscaling/v2
```

`k get nodes` shows the **kubelet's** version, not the control plane's. During an upgrade they differ.

---

## Exam angle

- **"What version is the cluster / node X running?":** `k version` for the API server, `k get nodes` for each kubelet.
- **"Which etcd / CoreDNS version is used?":** read the image tag of its Pod in `kube-system`.
- **"Upgrade to the next version":** that's one minor version up. `kubeadm upgrade plan` lists what's available (next chapters).
- **Trap:** reading the version from `k get nodes` as the cluster's version while workers are still on the old one.

## References

- [Releases](https://kubernetes.io/releases/) — supported versions, latest patches, end-of-life dates
- [Feature gates](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) — "Feature stages" (what alpha, beta and GA mean), and the table of gates with defaults
- [API overview](https://kubernetes.io/docs/reference/using-api/) — "API versioning" (`v1alpha1`, `v1beta1`, `v1`)
