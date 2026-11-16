# Resources: requests, limits, LimitRange, ResourceQuota

Chapter: [03-07 Resource requirements](../03-scheduling/07-resource-requirements.md). Units: [Resource units](../resource-units.md).

## Set requests and limits

```bash
kubectl set resources deployment web --requests=cpu=100m,memory=128Mi --limits=cpu=500m,memory=256Mi
kubectl set resources deployment web -c nginx --limits=memory=512Mi       # one container
kubectl set resources deployment web --limits=cpu=0,memory=0 --requests=cpu=0,memory=0   # remove
kubectl set resources -f pod.yaml --limits=cpu=200m --local -o yaml       # edit a file offline
kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[*].resources}{"\n"}'

# Bare Pod, in place (stable since v1.35)
kubectl patch pod web --subresource resize \
  --patch '{"spec":{"containers":[{"name":"web","resources":{"requests":{"cpu":"800m"},"limits":{"cpu":"800m"}}}]}}'
```

## Node capacity and usage

```bash
kubectl describe node node01 | grep -A6 -E "^Capacity|^Allocatable"
kubectl describe node node01 | grep -A8 "Allocated resources"   # sum of requests and limits
kubectl top node                                                 # actual usage, needs metrics-server
kubectl top pod -A --sort-by=memory
```

## Diagnose

```bash
kubectl describe pod web | grep -A5 Events                  # Insufficient cpu / memory
kubectl describe pod web | grep -A3 "Last State"            # Reason: OOMKilled, Exit Code: 137
kubectl get pod web -o jsonpath='{.status.qosClass}{"\n"}'  # Guaranteed / Burstable / BestEffort
kubectl describe rs -l app=web                              # FailedCreate from quota or LimitRange
```

## LimitRange

```bash
kubectl apply -f limit-range.yaml -n dev        # no generator: copy the YAML from the docs
kubectl get limitrange -n dev
kubectl describe limitrange cpu-resource-constraint -n dev
kubectl explain limitrange.spec.limits
```

## ResourceQuota

```bash
kubectl create quota my-quota -n dev \
  --hard=requests.cpu=4,requests.memory=4Gi,limits.cpu=10,limits.memory=10Gi,pods=10
kubectl create quota counts -n dev --hard=count/deployments.apps=5,services=3
kubectl describe quota -n dev                   # Used versus Hard
kubectl get resourcequota -A
```
