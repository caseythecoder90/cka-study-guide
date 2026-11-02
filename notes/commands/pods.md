# Pods

Creating, inspecting and deleting Pods. Chapter: [02-10 Pods](../02-core-concepts/10-pods.md).

## Create

```bash
kubectl run nginx --image=nginx                                  # Pod named exactly nginx
kubectl run nginx --image=nginx --port=80 --labels=app=web,tier=front
kubectl run nginx --image=nginx --env=MODE=prod
kubectl run busybox --image=busybox --restart=Never -- sleep 3600   # container args after --
kubectl run busybox --image=busybox --restart=Never --command -- sh -c 'echo hi; sleep 3600'   # override the command
kubectl run tmp --image=busybox --rm -it --restart=Never -- sh      # throwaway shell
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml # generate, edit, then:
kubectl apply -f pod.yaml
```

## Inspect

```bash
kubectl get pods
kubectl get pods -o wide                         # IP, NODE
kubectl get pods -w                              # watch status change
kubectl get pods --show-labels
kubectl describe pod nginx                       # containers, conditions, events
kubectl get pod nginx -o yaml
kubectl get pod nginx -o jsonpath='{.spec.containers[*].name}{"\n"}'   # container names
kubectl get pod nginx -o jsonpath='{.status.podIP}{"\n"}'
```

## Multi-container Pods

```bash
kubectl logs <pod> -c <container>
kubectl logs <pod> --all-containers
kubectl exec -it <pod> -c <container> -- sh
kubectl exec <pod> -c helper -- wget -qO- localhost:8080   # containers share localhost
```

## Delete

```bash
kubectl delete pod nginx
kubectl delete pod nginx --force --grace-period=0      # $now in commands/setup.md
```
