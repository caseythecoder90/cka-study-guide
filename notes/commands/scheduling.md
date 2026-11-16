# Scheduling

Placing Pods on nodes. Chapters: [03-01 Manual scheduling](../03-scheduling/01-manual-scheduling.md) · [03-03 Taints and tolerations](../03-scheduling/03-taints-and-tolerations.md) · [03-04 Node selectors](../03-scheduling/04-node-selectors.md) · [03-05 Node affinity](../03-scheduling/05-node-affinity.md).

## Is anything scheduling?

```bash
kubectl get pods -o wide                                # NODE <none> + Pending = not bound
kubectl describe pod nginx | grep -A3 Events            # <none> = no scheduler looked at it
kubectl get pods -n kube-system | grep scheduler        # is kube-scheduler running?
ls /etc/kubernetes/manifests/                           # on the control plane: kube-scheduler.yaml present?
kubectl get pod nginx -o jsonpath='{.spec.nodeName}{"\n"}'
kubectl get pod nginx -o jsonpath='{.spec.schedulerName}{"\n"}'   # default-scheduler
```

## Manual scheduling with nodeName

```bash
kubectl get nodes                                       # copy the exact node name
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx.yaml
# add under spec:   nodeName: node01
kubectl apply -f nginx.yaml

# Existing Pod: nodeName is immutable, so recreate it
kubectl get pod nginx -o yaml > nginx.yaml              # add nodeName under spec
kubectl replace --force -f nginx.yaml
kubectl get pod nginx -o wide                           # verify NODE and Running
```

## Binding an existing Pending Pod

```bash
cat > binding.json <<'EOF'
{"apiVersion":"v1","kind":"Binding","metadata":{"name":"nginx"},
 "target":{"apiVersion":"v1","kind":"Node","name":"node02"}}
EOF
kubectl create --raw /api/v1/namespaces/default/pods/nginx/binding -f binding.json

# The same with curl, through kubectl proxy for authentication
kubectl proxy &                                         # 127.0.0.1:8001
curl -X POST -H "Content-Type: application/json" --data @binding.json \
  http://127.0.0.1:8001/api/v1/namespaces/default/pods/nginx/binding
kill %1
```

## Taints on nodes

```bash
kubectl taint nodes node1 app=blue:NoSchedule
kubectl taint nodes node1 gpu:NoSchedule                       # no value
kubectl taint nodes node1 app=green:NoSchedule --overwrite     # change the value
kubectl taint nodes -l disktype=ssd app=blue:PreferNoSchedule  # by node label
kubectl taint nodes node1 app=blue:NoSchedule-                 # remove one
kubectl taint nodes node1 app-                                 # remove every taint with key app
kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-

kubectl describe node node1 | grep -A3 Taints
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

## Tolerations on Pods

```bash
kubectl run bee --image=nginx --dry-run=client -o yaml > bee.yaml
# add under spec (same level as containers):
#   tolerations:
#   - key: spray
#     operator: Equal
#     value: mortein
#     effect: NoSchedule
kubectl apply -f bee.yaml
kubectl get pod bee -o wide                                    # which node?
kubectl get pod bee -o jsonpath='{.spec.tolerations}{"\n"}'
kubectl describe pod mosquito | grep -A5 Events                # "untolerated taint"
```

## Node selectors

```bash
kubectl label nodes node01 size=Large
kubectl label nodes node01 size=Medium --overwrite
kubectl label nodes node01 size-
kubectl get nodes --show-labels
kubectl get nodes -L size                                      # as a column
kubectl get nodes -l size=Large                                # which nodes have it?

kubectl run myapp --image=nginx --dry-run=client -o yaml > myapp.yaml
# add under spec (same level as containers):
#   nodeSelector:
#     size: Large
kubectl apply -f myapp.yaml
kubectl get pod myapp -o wide
kubectl describe pod myapp | grep -A5 Events                   # Pending: no matching node
```

## Node affinity

```bash
kubectl explain pod.spec.affinity.nodeAffinity --recursive          # field names
kubectl explain pod.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms.matchExpressions

kubectl create deployment web --image=nginx --replicas=3 --dry-run=client -o yaml > web.yaml
# add under spec.template.spec (copy from the docs task page):
#   affinity:
#     nodeAffinity:
#       requiredDuringSchedulingIgnoredDuringExecution:
#         nodeSelectorTerms:
#         - matchExpressions:
#           - key: color
#             operator: In
#             values: ["blue"]
kubectl apply -f web.yaml
kubectl get pods -l app=web -o wide                                   # every replica on a blue node?
kubectl get nodes -L color
kubectl describe pod <pending-pod> | grep -A5 Events                  # didn't match node affinity/selector
kubectl rollout restart deployment web                                # reschedule after label changes
```
