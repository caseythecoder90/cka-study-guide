# 11 — Horizontal Pod Autoscaler (HPA)

An HPA changes the **replica count** of a workload to keep a metric near a target. It replaces watching `kubectl top pod` and running `kubectl scale` by hand ([10](10-autoscaling-overview.md)). The controller is part of the kube-controller-manager and checks every 15 seconds by default.

It can target anything with a `scale` subresource: Deployments, StatefulSets, ReplicaSets. Not DaemonSets.

**Two requirements for CPU and memory targets:**
1. **metrics-server is running.** `k top pods` must work.
2. **Every container in the Pod has a request** for that resource, if the target is a percentage. Sidecars included.

Runnable version: [`examples/hpa/hpa.yaml`](../../examples/hpa/hpa.yaml).

---

## 1. Creating one

```bash
k autoscale deploy my-app --cpu=50% --min=1 --max=10            # kubectl 1.34+
k autoscale deploy my-app --cpu-percent=50 --min=1 --max=10     # older kubectl; deprecated since 1.34
k autoscale deploy my-app --cpu=50% --min=1 --max=10 $do > hpa.yaml
k delete hpa my-app
```

**The `%` matters.** `--cpu=50%` means 50% of the request. `--cpu=50` means **50m** of CPU per Pod, a different kind of target.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app
spec:
  scaleTargetRef:                # which workload
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization        # percent of the REQUEST; AverageValue = absolute, e.g. 500m
        averageUtilization: 50
```

`autoscaling/v1` supports only a CPU percentage (`targetCPUUtilizationPercentage`). Use v2, stable since 1.23. (The slide says the HPA itself is built in since 1.23; it is much older than that.)

## 2. How it decides

**`desired = ceil(current replicas × current metric ÷ target)`**

A Pod requests 250m and limits to 500m. With a 50% target, the HPA aims for **125m** per Pod: the percentage is of the **request**, never the limit. At 450m, usage is 180%, so `ceil(1 × 180 ÷ 50)` = **4 replicas**.

- **Nothing changes within 10% of the target** (the default tolerance).
- **Several metrics:** it computes a count for each and uses the **highest**.
- **Scaling up has no waiting window. Scaling down waits 5 minutes** by default, using the highest count it recommended in that window. Tune it under `spec.behavior`:

```yaml
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 60     # default 300
      policies:
      - {type: Pods, value: 1, periodSeconds: 60}    # at most 1 Pod per minute
```

| Metric `type` | Source | API it needs |
|---|---|---|
| `Resource` | CPU/memory of the whole Pod | `metrics.k8s.io` (metrics-server) |
| `ContainerResource` | CPU/memory of **one** container | `metrics.k8s.io` |
| `Pods`, `Object` | In-cluster metrics, e.g. requests per second | `custom.metrics.k8s.io` (an adapter) |
| `External` | Outside the cluster, e.g. queue length, Datadog, Dynatrace | `external.metrics.k8s.io` (an adapter) |

## 3. Checking and troubleshooting

```bash
k get hpa                      # TARGETS  cpu: 30%/50%
k describe hpa my-app          # Conditions and Events say why
k get hpa my-app -w
```

| Symptom | Cause |
|---|---|
| `TARGETS` shows `<unknown>/50%` | metrics-server missing (`k top pods` fails), a container with no request, or Pods too new to have metrics |
| `AbleToScale False` | The HPA can't read or update the target's scale, e.g. a wrong name in `scaleTargetRef` |
| `ScalingLimited True` | Held at `minReplicas` or `maxReplicas` |
| Replicas rise, Pods `Pending` | The HPA worked; the nodes are full |
| Manual `k scale` is undone | Expected: the HPA owns the count |

**`kubectl autoscale` fails with "failed to create autoscaling/v2 HPA and the configuration is incompatible with autoscaling/v1".** This message (kubectl 1.34+) hides the real one. kubectl's v2 create request was refused, and then it tried to fall back to v1. `empty input` or `%!w(<nil>)` at the end is about the fallback, not the cause. Find the cause:

```bash
k auth can-i create hpa                   # RBAC
k api-versions | grep autoscaling         # autoscaling/v2 exists from 1.23
k autoscale deploy my-app --cpu=50% --max=10 --dry-run=server -v=1   # logs the real v2 error, creates nothing
```

## 4. Using it well

- **Remove `replicas` from a manifest you re-apply.** Otherwise `kubectl apply` resets the count and fights the HPA.
- **StatefulSets work**, but by default Pods are added and removed in order, and their PVCs stay after scaling down. The app must cope with members joining and leaving.
- **A sidecar can hide the app's load.** `Resource` utilization is total usage over total requests. `ContainerResource` targets the app container alone.
- **Memory is a poor target for many apps**, such as JVMs that don't give heap back: replicas go up and rarely come down.
- **Don't run HPA and VPA on the same CPU or memory metric.** They work against each other.

---

## Exam angle

- **"Autoscale Deployment X between A and B at N% CPU":** `k autoscale deploy X --min=A --max=B --cpu=N%`. If `--cpu` is unknown, use `--cpu-percent=N`. Verify with `k get hpa`.
- **"...with a scale-down window of S seconds":** generate the YAML, then add `behavior.scaleDown.stabilizationWindowSeconds`. Recall: `kubectl explain hpa.spec.behavior.scaleDown`.
- **`<unknown>` target:** check `k top pods`, then the requests.
- **Traps:** a missing `%`; the wrong HPA name (`--name` sets it; the default is the workload's name); no requests on the target.

## References

- [HorizontalPodAutoscaler walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) — "Create the HorizontalPodAutoscaler", and "Autoscaling on multiple metrics and custom metrics" (v2 YAML)
- [Horizontal Pod autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) — "Algorithm details", "Configurable scaling behavior" (`behavior` YAML), "Container resource metrics"
- [kubectl autoscale](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_autoscale/) — "Examples"
