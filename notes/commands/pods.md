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

## Manifests

Chapter: [02-11 Pods with YAML](../02-core-concepts/11-pods-with-yaml.md).

```bash
kubectl run myapp-pod --image=nginx --labels=app=myapp,type=front-end --dry-run=client -o yaml > pod.yaml
kubectl apply -f pod.yaml --dry-run=server       # full validation by the API server, stores nothing
kubectl apply -f pod.yaml                        # create or update
kubectl create -f pod.yaml                       # errors with AlreadyExists on a rerun
kubectl replace --force -f pod.yaml              # delete and recreate
kubectl api-resources                            # APIVERSION and KIND for everything served
kubectl api-resources | grep -i deployment
kubectl explain pod.spec.containers              # fields, types, -required-
kubectl explain pod.spec.containers.env
kubectl explain pod --recursive | less
```
