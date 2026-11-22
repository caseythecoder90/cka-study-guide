# Scheduler profiles

Chapter: [03-12 Scheduler Profiles](../03-scheduling/12-scheduler-profiles.md).

## Inspect

```bash
kubectl -n kube-system get pod -l component=kube-scheduler -o yaml | grep -- --config   # config file in use?
cat /etc/kubernetes/manifests/kube-scheduler.yaml                                       # on the control plane
kubectl get events -o wide | grep Scheduled                                             # SOURCE = profile name
kubectl -n kube-system logs <kube-scheduler-pod>
```

## Add profiles to the kubeadm scheduler

```bash
sudo vi /etc/kubernetes/scheduler-config.yaml   # KubeSchedulerConfiguration with clientConnection.kubeconfig + profiles
sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml
#   command:      - --config=/etc/kubernetes/scheduler-config.yaml
#   volumeMounts: - {name: sched-config, mountPath: /etc/kubernetes/scheduler-config.yaml, readOnly: true}
#   volumes:      - {name: sched-config, hostPath: {path: /etc/kubernetes/scheduler-config.yaml, type: FileOrCreate}}
kubectl -n kube-system get pod -l component=kube-scheduler -w
```

## Use a profile

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx.yaml
# add under spec:   schedulerName: no-scoring-scheduler
kubectl apply -f nginx.yaml
```
