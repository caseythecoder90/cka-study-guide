# Rollout commands

Chapter: [05-01 Rolling updates and rollbacks](../05-application-lifecycle-management/01-rolling-updates-and-rollbacks.md). Deployment basics: [`deployments.md`](deployments.md).

## Update

```bash
kubectl get deploy <d> -o jsonpath='{.spec.template.spec.containers[*].name}'
kubectl set image deploy/<d> <container>=<image>
kubectl annotate deploy/<d> kubernetes.io/change-cause="<why>"    # replaces deprecated --record
kubectl rollout status deploy/<d>
kubectl get rs -l app=<label>                                     # new RS up, old at 0
kubectl get deploy <d> -o jsonpath='{..image}'                    # what is live
```

## History and rollback

```bash
kubectl rollout history deploy/<d>
kubectl rollout history deploy/<d> --revision=N
kubectl rollout undo deploy/<d>                     # previous revision; renumbered as the newest
kubectl rollout undo deploy/<d> --to-revision=N
```

## Strategy

```bash
kubectl describe deploy <d> | grep -A1 StrategyType
kubectl explain deploy.spec.strategy.rollingUpdate
kubectl edit deploy <d>        # Recreate: set type and delete the rollingUpdate block
```
