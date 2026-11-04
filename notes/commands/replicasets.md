# ReplicaSets

Creating, inspecting, scaling and repairing ReplicaSets. Chapter: [02-12 ReplicaSets](../02-core-concepts/12-replicasets.md).

## Create

```bash
kubectl apply -f replicaset-definition.yaml      # rerunnable; create -f errors if it exists
kubectl create deployment myapp --image=nginx --replicas=3 --dry-run=client -o yaml > rs.yaml
# In rs.yaml: change kind to ReplicaSet and delete the strategy line
```

## Inspect

```bash
kubectl get rs                                   # DESIRED, CURRENT, READY
kubectl get rs -o wide                           # + CONTAINERS, IMAGES, SELECTOR
kubectl get rc                                   # legacy ReplicationControllers
kubectl describe rs myapp-replicaset             # Pod counts and events
kubectl get pods --show-labels
kubectl get pods -l type=front-end
kubectl get pod <pod> -o jsonpath='{.metadata.ownerReferences[0].name}{"\n"}'   # owner
```

## Scale

```bash
kubectl scale --replicas=6 replicaset myapp-replicaset
kubectl scale --replicas=6 rs/myapp-replicaset
kubectl scale --replicas=6 -f replicaset-definition.yaml   # the file still has the old value
kubectl edit rs myapp-replicaset                 # spec.replicas
```

## Change the Pods

```bash
kubectl edit rs myapp-replicaset                 # fix the template
kubectl delete pod -l type=front-end             # replacements use the new template
kubectl get pods -w
```

## Delete

```bash
kubectl delete rs myapp-replicaset               # also deletes its Pods
kubectl delete rs myapp-replicaset --cascade=orphan   # keeps the Pods
```
