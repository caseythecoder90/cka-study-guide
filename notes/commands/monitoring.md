# Monitoring commands

Chapter: [04-01 Monitoring cluster components](../04-logging-and-monitoring/01-monitoring-cluster-components.md)

## Install and check metrics-server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl -n kube-system get deploy metrics-server
kubectl get apiservice | grep metrics               # AVAILABLE must be True
kubectl -n kube-system logs deploy/metrics-server
kubectl -n kube-system edit deploy metrics-server   # lab clusters: add --kubelet-insecure-tls to args
```

## Read usage

```bash
kubectl top node [--sort-by=cpu|memory] [--show-capacity]
kubectl top pod [-n ns | -A] [-l key=value] [--containers] [--sort-by=cpu|memory] [--sum]
kubectl top pod -A --sort-by=memory --no-headers | head -1    # heaviest Pod
kubectl describe node <node>          # requests/limits (Allocated resources), not usage
```
