# Namespaces

Creating and using namespaces, names across them, and quotas. Chapter: [02-17 Namespaces](../02-core-concepts/17-namespaces.md).

## Create and list

```bash
kubectl create namespace dev
kubectl create namespace dev --dry-run=client -o yaml > ns.yaml
kubectl get namespaces                           # short name: ns
kubectl describe namespace dev                   # labels, quotas, limit ranges
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```

## Work in one

```bash
kubectl get pods -n kube-system
kubectl get pods -A                              # --all-namespaces
kubectl get pods -A | grep <name>
kubectl apply -f pod.yaml -n dev
kubectl config set-context --current --namespace=dev
kubectl config view --minify | grep namespace    # confirm
kubectl config set-context --current --namespace=default
```

## Names across namespaces

```bash
kubectl exec <pod> -n dev -- cat /etc/resolv.conf                                    # search list
kubectl run tmp --image=busybox --rm -it --restart=Never -n dev -- nslookup db-service
kubectl run tmp --image=busybox --rm -it --restart=Never -- nslookup db-service.dev
kubectl run tmp --image=busybox --rm -it --restart=Never -- nslookup db-service.dev.svc.cluster.local
kubectl run tmp --image=busybox --rm -it --restart=Never -- nslookup web-0.nginx.prod   # StatefulSet Pod
```

## Quotas

```bash
kubectl create quota compute-quota -n dev \
  --hard=pods=10,requests.cpu=4,requests.memory=5Gi,limits.cpu=10,limits.memory=10Gi
kubectl get quota -n dev
kubectl describe quota -n dev                    # Used versus Hard
```

## Delete

```bash
kubectl delete namespace dev                     # deletes everything in it
```
