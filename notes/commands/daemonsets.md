# DaemonSets

Chapter: [03-08 DaemonSets](../03-scheduling/08-daemonsets.md).

## Inspect

```bash
kubectl get daemonsets -A                       # DESIRED, CURRENT, READY, NODE SELECTOR
kubectl get ds -n kube-system                   # kube-proxy, CNI agents
kubectl describe ds kube-proxy -n kube-system
kubectl get pods -n kube-system -l k8s-app=kube-proxy -o wide   # one per node
kubectl get ds kube-proxy -n kube-system -o jsonpath='{.spec.template.spec.tolerations}{"\n"}'
kubectl get pod <daemon-pod> -o yaml | grep -A8 nodeAffinity    # matchFields metadata.name
```

## Create (no generator)

```bash
kubectl create deployment monitoring-daemon --image=nginx --dry-run=client -o yaml > ds.yaml
# edit ds.yaml:
#   kind: Deployment  ->  kind: DaemonSet
#   delete: replicas, strategy: {}, status: {}
#   optional: tolerations for node-role.kubernetes.io/control-plane, a nodeSelector
kubectl apply -f ds.yaml
kubectl get ds monitoring-daemon
kubectl get pods -l app=monitoring-daemon -o wide
kubectl get nodes                               # compare: DESIRED should equal eligible nodes
```

## Update and roll back

```bash
kubectl set image ds/monitoring-daemon nginx=nginx:1.27
kubectl rollout status ds/monitoring-daemon
kubectl rollout history ds/monitoring-daemon
kubectl rollout undo ds/monitoring-daemon
kubectl rollout restart ds/monitoring-daemon
kubectl delete ds monitoring-daemon             # also deletes its Pods
```
