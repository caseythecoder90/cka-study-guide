# Scaling commands

Chapter: [05-10 Autoscaling overview](../05-application-lifecycle-management/10-autoscaling-overview.md). Resources in detail: [`resources.md`](resources.md).

```bash
k scale deploy web --replicas=5
k scale deploy web --current-replicas=3 --replicas=5
k set resources deploy web --requests=cpu=200m,memory=256Mi --limits=memory=512Mi   # starts a rollout
kubeadm token create --print-join-command      # run the printed command on the new node
k get hpa                                      # an HPA overrides a manual scale
k describe pod <pending-pod>                   # Insufficient cpu / memory
```

## HorizontalPodAutoscaler

Chapter: [05-11](../05-application-lifecycle-management/11-horizontal-pod-autoscaler.md). Example: [`examples/hpa/`](../../examples/hpa/).

```bash
k autoscale deploy my-app --cpu=50% --min=1 --max=10           # 1.34+; older: --cpu-percent=50
k get hpa ; k describe hpa my-app
k auth can-i create hpa
k autoscale deploy my-app --cpu=50% --max=10 --dry-run=server -v=1   # show the real error, create nothing
k explain hpa.spec.behavior.scaleDown
k delete hpa my-app
```

## In-place Pod resize

Chapter: [05-12](../05-application-lifecycle-management/12-in-place-pod-resize.md). Example: [`examples/resize/`](../../examples/resize/).

```bash
k edit pod resize-demo --subresource resize
k get pod resize-demo -o jsonpath='{.spec.containers[0].resources}{"\n"}{.status.containerStatuses[0].resources}{"\n"}'
k describe pod resize-demo            # PodResizePending (Deferred / Infeasible), PodResizeInProgress
```

## VerticalPodAutoscaler

Chapter: [05-13](../05-application-lifecycle-management/13-vertical-pod-autoscaler.md). Example: [`examples/vpa/`](../../examples/vpa/). No imperative create command.

```bash
k get pods -n kube-system | grep vpa      # recommender, updater, admission-controller
k get crd | grep verticalpodautoscalers
k get vpa ; k describe vpa my-app-vpa     # Recommendation: Target, Lower / Upper Bound
k explain vpa.spec.updatePolicy
```
