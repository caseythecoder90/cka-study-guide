# 01 — OS Upgrades: Drain, Cordon, Uncordon

To patch, upgrade or reboot a node, first move its workloads off, then do the work, then let Pods come back. Three commands:

| Command | Marks the node unschedulable | Removes running Pods |
|---|---|---|
| `kubectl cordon node-1` | Yes | No |
| `kubectl drain node-1` | Yes (it cordons first) | **Yes**: evicts them |
| `kubectl uncordon node-1` | Removes the mark | No: nothing moves back |

![Two timelines. Planned: kubectl drain cordons and evicts, controllers recreate Pods on other nodes, the node is patched, kubectl uncordon. Unplanned: the node stops reporting, about 50 seconds later it is Ready=Unknown and tainted unreachable, 300 seconds after that Pods are evicted and recreated by their controllers, while bare Pods are lost](./diagrams/01-node-down-vs-drain.svg)

---

## 1. When a node just goes down

Without a drain, Kubernetes waits before giving up on a node ([02-06](../02-core-concepts/06-kube-controller-manager.md)):

1. **About 50 seconds** after the kubelet stops reporting, the node becomes `NotReady` (its `Ready` condition is `Unknown`) and is tainted `node.kubernetes.io/unreachable:NoExecute`.
2. Every Pod tolerates that taint for **300 seconds** by default, through an automatic `tolerationSeconds: 300` ([03-03](../03-scheduling/03-taints-and-tolerations.md)). Then it is evicted.
3. Pods owned by a controller (Deployment, ReplicaSet, StatefulSet) are recreated elsewhere. **A bare Pod is simply gone.**

If the node is back within those 5 minutes, its Pods start again in place. If it's back later, it comes back empty.

The course still calls this the **pod eviction timeout** on the controller manager. That flag (`--pod-eviction-timeout`) has been removed; the 5 minutes now comes from the Pods' tolerations. A Pod can set its own, such as `tolerationSeconds: 3600` to wait an hour.

Rebooting without draining is a gamble on coming back within 5 minutes. **Draining is the safe way.**

## 2. Draining

```bash
k drain node-1 --ignore-daemonsets           # the form you almost always need
k get nodes                                  # node-1   Ready,SchedulingDisabled
k get pods -A -o wide | grep node-1          # only DaemonSet and static Pods remain
# ... patch and reboot node-1 ...
k uncordon node-1                            # Ready
```

**What drain does:**
- It **evicts** Pods. They aren't moved: they're terminated gracefully, and their controllers create replacements on other nodes.
- Eviction respects **PodDisruptionBudgets**. A PDB that allows no disruption makes drain retry until its `--timeout`, or forever by default.
- It cordons the node first, using the taint `node.kubernetes.io/unschedulable:NoSchedule`.

**Drain refuses to start** while any of these Pods are on the node. The error names them and the flag to use:

| Pod on the node | Flag | What the flag does |
|---|---|---|
| DaemonSet Pods | `--ignore-daemonsets` | Leaves them running (deleting them is pointless: the DaemonSet would recreate them) |
| Pods with no controller (bare Pods) | `--force` | Deletes them, **permanently**: nothing recreates them |
| Pods using an `emptyDir` | `--delete-emptydir-data` | Evicts them; the `emptyDir` data is lost |

Static Pods are skipped: drain can't remove them, because the kubelet runs them from a file on the node ([03-09](../03-scheduling/09-static-pods.md)).

Other flags: `--pod-selector` to drain only some Pods, `--grace-period` and `--timeout`, and `--disable-eviction` to delete instead of evict (this skips PDBs).

## 3. After the work

- **`uncordon` doesn't rebalance.** Pods that were evicted stay where they were recreated. Only Pods created later can land on `node-1`.
- **`cordon` alone** is for "no new Pods here", leaving the running ones in place, for example before a drain you'll do later.
- A node left cordoned by mistake shows `SchedulingDisabled`. New Pods go elsewhere or stay `Pending`.

---

## Exam angle

- **"Node X needs maintenance: move its workloads off and make it unschedulable":** `k drain X --ignore-daemonsets`. Verify with `k get nodes` (`SchedulingDisabled`) and `k get pods -o wide`.
- **"Drain fails: cannot delete Pods not managed by a controller":** that's a bare Pod. Decide whether the task allows losing it, then add `--force`. If it must survive, recreate it on another node first.
- **"Maintenance done":** `k uncordon X`. Pods don't move back by themselves.
- **"Stop new Pods on X but leave running ones":** `k cordon X`.
- **Traps:** forgetting to uncordon; running drain on the wrong context; draining the node your only replica runs on, with a PDB that blocks it.

## References

- [Safely drain a node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/) — "Use kubectl drain to remove a node from service", "The Eviction API"
- [kubectl drain](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_drain/) — every flag
- [Taints and tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) — "Taint based Evictions" (the 300-second default)
