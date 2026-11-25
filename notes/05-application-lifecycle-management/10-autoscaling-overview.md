# 10 — Autoscaling Overview

Scaling answers two separate questions:
- **What grows?** The workload (Pods) or the cluster (nodes).
- **How?** **Horizontally**, by adding more of them, or **vertically**, by making each one bigger.

| | Workload (Pods) | Cluster (nodes) |
|---|---|---|
| **Horizontal:** more | More replicas | More nodes |
| by hand | `kubectl scale` | Provision a machine, then `kubeadm join` |
| automatic | **HorizontalPodAutoscaler (HPA)** | **Cluster Autoscaler** or Karpenter |
| **Vertical:** bigger | Larger CPU/memory requests and limits | A larger machine |
| by hand | `kubectl edit`, `kubectl set resources` | Replace the node: drain it, remove it, join a bigger one |
| automatic | **VerticalPodAutoscaler (VPA)** | Uncommon: autoscalers add and remove nodes instead |

HPA and VPA are the exam topics, and they get their own chapters.

---

## 1. What is built in

| Component | Ships with Kubernetes? | Acts on |
|---|---|---|
| HPA | **Yes:** the `autoscaling/v2` API, run by the kube-controller-manager | CPU/memory usage or custom metrics. Changes `replicas` |
| VPA | **No:** an add-on with its own CRDs and controllers | Usage over time. Changes Pod `resources` |
| Cluster Autoscaler, Karpenter | **No:** add-ons that need a cloud provider integration | **Pods that can't be scheduled**, and underused nodes |

- **HPA on CPU or memory, and VPA, need metrics-server** ([04-01](../04-logging-and-monitoring/01-monitoring-cluster-components.md)). If `kubectl top pods` fails, they have no data.
- **Node autoscalers look at Pod requests, not at node usage.** A node at 95% CPU doesn't trigger one; a Pod left `Pending` for lack of CPU or memory does.

They work in a chain: load rises → HPA adds replicas → some don't fit and stay `Pending` → the node autoscaler adds a node → the scheduler places them.

## 2. Scaling by hand

```bash
# horizontal
k scale deploy web --replicas=5
k scale deploy web --current-replicas=3 --replicas=5     # only if it is at 3 now
k get deploy web                                         # READY 5/5

# vertical
k set resources deploy web --requests=cpu=200m,memory=256Mi --limits=memory=512Mi
k edit deploy web                                        # same, in the editor

# more nodes: on a control-plane node, print the command to run on the new machine
kubeadm token create --print-join-command
k get nodes
```

- **`scale` adds or removes Pods and starts no rollout.** It works on Deployments, ReplicaSets, StatefulSets and ReplicationControllers, but not DaemonSets.
- **Changing resources in a Deployment does start a rollout:** the template changed, so every Pod is replaced ([01](01-rolling-updates-and-rollbacks.md)).
- A running Pod's CPU and memory can also be resized **in place**, through its `resize` subresource (stable since 1.35, [12](12-in-place-pod-resize.md)).

---

## Exam angle

- **"Scale Deployment X to N":** `k scale deploy X --replicas=N`, then check READY.
- **New replicas stay `Pending`:** scaling worked, but there's no room. `describe pod` shows `Insufficient cpu` or `Insufficient memory`. Lower the requests or free up a node.
- **Deployment returns to its old size after `scale`:** an HPA owns the replica count. Check `k get hpa`.
- **Resources on a Deployment:** `k set resources` is faster than editing YAML.

## References

- [Autoscaling workloads](https://kubernetes.io/docs/concepts/workloads/autoscaling/) — "Scaling workloads horizontally", "Scaling workloads vertically"
- [Node autoscaling](https://kubernetes.io/docs/concepts/cluster-administration/node-autoscaling/) — Cluster Autoscaler versus Karpenter
- [kubectl scale](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_scale/) — "Examples"
