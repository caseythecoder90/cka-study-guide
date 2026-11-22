# 01 — Monitoring Cluster Components

What you want to know about a cluster:

- **Node level:** how many nodes there are, which are healthy, and their CPU, memory, network and disk use.
- **Pod level:** how many Pods there are and each one's CPU and memory use.

Kubernetes ships no full monitoring stack. The built-in piece is **metrics-server**, which gives only *current* CPU and memory figures. Anything with history, dashboards or alerts is a separate product:
- open source: Prometheus with Grafana, or the Elastic Stack;
- commercial: Datadog or Dynatrace.

(Heapster, the original tool, is retired. Ignore old guides that use it.)

---

## 1. How the numbers are collected

![Containers report cgroup stats to the kubelet and its built-in cAdvisor; metrics-server scrapes each kubelet's /metrics/resource endpoint, keeps the latest values in memory and is registered with the API server as the metrics.k8s.io APIService, which kubectl top, HPA and VPA read](./diagrams/01-resource-metrics-pipeline.svg)

1. **cAdvisor**, built into the kubelet, reads each container's CPU and memory from the node's cgroups. The kubelet serves the results at `/metrics/resource` on port 10250.
2. **metrics-server** (one per cluster, a Deployment in `kube-system`) scrapes every kubelet and keeps **only the latest values, in memory**. It writes nothing to disk, so there is no history.
3. It is registered as an **APIService** for `metrics.k8s.io`. The API server forwards those requests to it (the *aggregation layer*), so `kubectl top`, the HorizontalPodAutoscaler and the VerticalPodAutoscaler can ask the API server for metrics.

Its own README warns that metrics-server is meant for autoscaling, not monitoring. Use Prometheus or a commercial tool for that.

## 2. Installing metrics-server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
# minikube: minikube addons enable metrics-server
```

**Slide correction:** the lecture's `git clone` followed by `kubectl create -f deploy/1.8+/` no longer works. The repo has no `deploy/` directory; use the release manifest above.

On lab clusters where the kubelet certificates aren't signed by the cluster CA, the Pod runs but never becomes Ready, and its logs show x509 errors. The fix is to add `--kubelet-insecure-tls` to the container args. The README marks it "for testing purposes only".

```bash
kubectl -n kube-system edit deploy metrics-server   # add - --kubelet-insecure-tls under args
```

Check that it works, in this order:

```bash
kubectl -n kube-system get deploy metrics-server        # READY 1/1
kubectl get apiservice | grep metrics                   # AVAILABLE True
kubectl -n kube-system logs deploy/metrics-server       # x509 / connection errors
kubectl top node                                        # works after about a minute
```

`error: Metrics API not available` means one of the checks above is failing.

## 3. Reading the metrics

```bash
kubectl top node                          # CPU (cores, m = millicores), CPU%, memory, memory%
kubectl top node --sort-by=cpu
kubectl top pod                           # current namespace
kubectl top pod -A --sort-by=memory       # whole cluster, heaviest first
kubectl top pod -n prod -l app=web --containers   # per container
kubectl top pod --sum                     # adds a total row
```

- `--sort-by` takes only `cpu` or `memory`.
- The percentages in `top node` are measured against the node's **allocatable** resources. Add `--show-capacity` to measure against total capacity.

To find the heaviest Pod by script, ask for no headers and take the first line:

```bash
kubectl top pod -A --sort-by=cpu --no-headers | head -1
```

---

## Exam angle

- **"Which node or Pod uses the most CPU or memory? Write its name to /opt/answer.txt."** Use `kubectl top ... --sort-by=cpu|memory --no-headers | head -1`. Check the namespace: add `-A` if the task says cluster-wide, or `-n <ns>` if it names one. Write only the name unless asked otherwise.
- **"Metrics API not available":** check the Deployment, then `get apiservice`, then the Pod's logs. A lab cluster usually needs `--kubelet-insecure-tls`.
- **Just installed:** `top` returns nothing for about a minute until the first scrape. Wait, don't reinstall.
- **Trap:** `top` without `--containers` shows Pod totals, so a per-container question needs the flag. Also, `top` shows *usage*, not requests or limits. For requests, use `kubectl describe node` ("Allocated resources").

## References

- [Resource metrics pipeline](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/) — "Metrics Server": how metrics flow from the kubelet through the aggregation layer
- [kubectl top pod](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_top/kubectl_top_pod/) and [kubectl top node](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_top/kubectl_top_node/) — every flag
- [metrics-server README](https://github.com/kubernetes-sigs/metrics-server) — the install command and the list of useful flags, including `--kubelet-insecure-tls`

## Related notes

- CKAD: [Monitoring and metrics server](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/04-observability/05-monitoring-metrics-server.md) — the developer's view
