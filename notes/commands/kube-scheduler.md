# kube-scheduler

Finding and fixing the scheduler, and reading its decisions. Chapter: [02-07 kube-scheduler](../02-core-concepts/07-kube-scheduler.md). Section 3 adds the scheduling inputs (taints, affinity, requests, manual scheduling).

## Where its options are

```bash
kubectl -n kube-system get pods | grep scheduler                # kubeadm: kube-scheduler-<node>
cat /etc/kubernetes/manifests/kube-scheduler.yaml
cat /etc/systemd/system/kube-scheduler.service                  # from scratch
cat /etc/kubernetes/config/kube-scheduler.yaml                  # the --config file, if one is used
ps -aux | grep [k]ube-scheduler | tr ' ' '\n' | grep -- '^--'
kubectl -n kube-system get lease kube-scheduler -o jsonpath='{.spec.holderIdentity}{"\n"}'   # active leader
```

## Reading its decisions

```bash
kubectl get pods -o wide                                        # NODE column
kubectl get pods -A --field-selector spec.nodeName=             # not scheduled yet
kubectl get pods -A --field-selector status.phase=Pending
kubectl describe pod <pod> | grep -A5 Events                    # Scheduled / FailedScheduling
kubectl get events -A --field-selector reason=FailedScheduling
kubectl get pod <pod> -o jsonpath='{.spec.schedulerName}{"\n"}' # default-scheduler
kubectl describe node <node> | grep -A8 "Allocated resources"   # requests already placed on a node
```

## When it is down (Pods Pending, no node)

```bash
crictl ps -a | grep kube-scheduler
crictl logs <container-id>
cp /etc/kubernetes/manifests/kube-scheduler.yaml /root/kube-scheduler.yaml.bak   # backup OUTSIDE manifests/
vi /etc/kubernetes/manifests/kube-scheduler.yaml                # fix binary name, flag, kubeconfig path
watch crictl ps
kubectl get pods -o wide                                        # Pending Pods get a node once it is back
```

## Installing the binary

```bash
V=$(curl -sL https://dl.k8s.io/release/stable.txt)
wget https://dl.k8s.io/release/${V}/bin/linux/amd64/kube-scheduler
```
