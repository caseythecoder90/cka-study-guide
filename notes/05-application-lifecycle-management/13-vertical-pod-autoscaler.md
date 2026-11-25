# 13 — Vertical Pod Autoscaler (VPA)

A VPA watches how much CPU and memory a workload uses and sets its Pods' **requests** (and, by default, limits) to match. An HPA changes how many Pods there are; a VPA changes how big each one is.

It is **not built in**: it's an add-on with its own CRD (`autoscaling.k8s.io/v1`), and it needs metrics-server.

Runnable version: [`examples/vpa/vpa.yaml`](../../examples/vpa/vpa.yaml).

---

## 1. What happens when you change resources by hand

It depends on what you edit:

| You edit | What happens to running Pods |
|---|---|
| Deployment template | Rolling update: new Pods are created, and old ones removed as the new ones become ready ([01](01-rolling-updates-and-rollbacks.md)) |
| StatefulSet template | Pods are **deleted and recreated one at a time**, highest number first, keeping name and PVC. With `OnDelete`, nothing until you delete them |
| DaemonSet template | Rolling update, node by node. With `OnDelete`, nothing until you delete them |
| **ReplicaSet** template | **Nothing.** Only Pods created later get the new values |
| **Bare Pod** | **The edit is refused**, unless you use `--subresource resize` ([12](12-in-place-pod-resize.md)) |

## 2. Installing and checking

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler && ./hack/vpa-up.sh

k get pods -n kube-system | grep vpa      # 3 components
k get crd | grep verticalpodautoscalers
```

The single-file URL on the slide (`.../releases/latest/download/vertical-pod-autoscaler.yaml`) returns 404.

| Component | Job |
|---|---|
| **Recommender** | Reads usage (metrics API plus history) and writes a recommendation into the VPA's `status` |
| **Updater** | Finds Pods whose requests are too far from the recommendation, and **evicts** them or resizes them in place |
| **Admission controller** | A webhook that puts the recommended values into each **new** Pod as it's created |

So: Recommender suggests → Updater evicts the Pod → the Deployment creates a replacement → the admission controller sets its resources. **The Deployment's template never changes**; only the Pods do.

## 3. The manifest

```yaml
apiVersion: autoscaling.k8s.io/v1       # not autoscaling/v2: that's the HPA
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:                            # the HPA calls this scaleTargetRef
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Recreate"
  resourcePolicy:
    containerPolicies:
    - containerName: my-app             # "*" = every container
      minAllowed: {cpu: 250m}
      maxAllowed: {cpu: "2"}
      controlledResources: ["cpu"]      # default: cpu and memory
      controlledValues: RequestsAndLimits   # default; RequestsOnly leaves limits alone
```

| `updateMode` | New Pods | Running Pods |
|---|---|---|
| `Off` | Unchanged | Unchanged: recommendations only |
| `Initial` | Set | Unchanged |
| `Recreate` (default) | Set | Evicted and recreated |
| `InPlaceOrRecreate` | Set | Resized in place; evicted if that fails. Needs Kubernetes 1.33+ |
| `InPlace` | Set | Resized in place, never evicted. Alpha, behind a feature gate |
| `Auto` | Set | **Deprecated**: same as `Recreate` |

`Off` is the safe way to start: read the recommendation, then set the requests yourself.

## 4. Reading it

```bash
k get vpa
k describe vpa my-app-vpa          # Status > Recommendation
```

| Field | Meaning |
|---|---|
| `Target` | What the VPA sets as the request |
| `Lower Bound` / `Upper Bound` | A Pod whose request is outside this range gets updated |
| `Uncapped Target` | The target if `minAllowed`/`maxAllowed` didn't apply |

**No recommendation:** check that metrics-server works (`k top pods`), the recommender is running, and `targetRef` is right. Give it a few minutes of data.

**A recommendation, but Pods don't change:** check `updateMode`. Also, the updater won't evict a workload with fewer than **2 live replicas** by default, or when a PodDisruptionBudget forbids it. The lecture's 1-replica Deployment would not be updated. `updatePolicy.minReplicas: 1` overrides this.

**Pods `Pending` afterwards:** the recommendation doesn't fit on any node. Set `maxAllowed`.

## 5. VPA or HPA

| | VPA | HPA |
|---|---|---|
| Changes | Requests and limits per Pod | Replica count |
| Running Pods | Evicted, or resized in place | Left alone |
| Traffic spikes | Poor: reacts slowly and may evict | Good, within its 15 s check and Pod start-up time |
| Best for | Databases, apps that can't run more replicas, finding the right requests | Stateless services |

**Never both on the same metric.** A VPA raising requests lowers the HPA's utilization percentage, so they fight. HPA on CPU with VPA on memory only is fine.

---

## Exam angle

- **"Create a VPA for Deployment X in mode M, with min/max":** there is no imperative command. Copy the YAML from the docs page below and change `targetRef`, `updateMode` and `minAllowed`/`maxAllowed`. Verify with `k describe vpa`.
- **"What CPU does the VPA recommend?":** `k describe vpa X`, the `Target` value.
- **`no matches for kind "VerticalPodAutoscaler"`:** the CRD isn't installed, or the `apiVersion` is wrong.
- **Traps:** `scaleTargetRef` instead of `targetRef`; `autoscaling/v1` instead of `autoscaling.k8s.io/v1`; looking for the new values in the Deployment instead of the Pods.

## References

- [Vertical Pod Autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/vertical-pod-autoscale/) — "Update modes" and "Resource policies" (YAML to copy)
- [VPA installation](https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/docs/installation.md) — install command and version compatibility
- [VPA known limitations](https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/docs/known-limitations.md) — HPA conflicts, Pending Pods
