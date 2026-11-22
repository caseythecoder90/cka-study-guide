# Multiple schedulers

Chapter: [03-11 Multiple Schedulers](../03-scheduling/11-multiple-schedulers.md).

## Inspect

```bash
kubectl -n kube-system get pods                                          # schedulers running
kubectl -n kube-system get pod -l component=kube-scheduler -o jsonpath='{..image}{"\n"}'   # image to reuse
kubectl get pod nginx -o jsonpath='{.spec.schedulerName}{"\n"}'
kubectl get events -o wide | grep Scheduled                             # SOURCE = scheduler that bound it
kubectl -n kube-system logs my-scheduler
```

## Deploy a second scheduler

```bash
cat > my-scheduler-config.yaml <<'EOT'
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-scheduler
leaderElection:
  leaderElect: false
EOT
kubectl -n kube-system create configmap my-scheduler-config --from-file=my-scheduler-config.yaml
# Pod in kube-system: command kube-scheduler --config=<mountPath>/my-scheduler-config.yaml,
# ConfigMap volume at <mountPath>, serviceAccountName with system:kube-scheduler binding
kubectl apply -f my-scheduler.yaml
```

## Use it

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx.yaml
# add under spec:   schedulerName: my-scheduler
kubectl apply -f nginx.yaml
kubectl replace --force -f nginx.yaml     # existing Pod: schedulerName is immutable
```
