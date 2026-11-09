# Scheduling

Placing Pods on nodes. Chapter: [03-01 Manual scheduling](../03-scheduling/01-manual-scheduling.md).

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
