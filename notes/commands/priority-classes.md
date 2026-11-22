# PriorityClasses

Chapter: [03-10 PriorityClasses](../03-scheduling/10-priority-classes.md).

## Inspect

```bash
kubectl get priorityclass                         # short name: pc; VALUE, GLOBAL-DEFAULT, PREEMPTIONPOLICY
kubectl describe pc system-node-critical
kubectl get pods -A -o custom-columns=NAME:.metadata.name,CLASS:.spec.priorityClassName,PRIORITY:.spec.priority
kubectl get pod nginx -o jsonpath='{.spec.priorityClassName}{" "}{.spec.priority}{"\n"}'
kubectl explain priorityclass                     # top-level fields, no spec
```

## Create

```bash
kubectl create priorityclass high-priority --value=1000000000 --description="Mission-critical Pods"
kubectl create priorityclass batch-wait --value=500000 --preemption-policy=Never
kubectl create priorityclass default-prio --value=1000 --global-default=true
kubectl create priorityclass high-priority --value=1000000000 --dry-run=client -o yaml > pc.yaml
```

## Use it

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
# add under spec:   priorityClassName: high-priority
kubectl apply -f pod.yaml

kubectl get pod nginx -o yaml > nginx.yaml        # running Pod: field is not editable
# add priorityClassName, then:
kubectl replace --force -f nginx.yaml

kubectl patch deployment web -p '{"spec":{"template":{"spec":{"priorityClassName":"high-priority"}}}}'
kubectl rollout status deployment/web
```

## Preemption

```bash
kubectl get pod big -o jsonpath='{.status.nominatedNodeName}{"\n"}'   # node it is preempting on
kubectl get events --field-selector reason=Preempted
kubectl describe pod big | tail                   # FailedScheduling / preemption messages
```
