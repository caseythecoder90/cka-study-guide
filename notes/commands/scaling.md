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
