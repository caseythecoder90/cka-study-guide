# kube-controller-manager

Finding, reading and fixing the controller manager, and watching the node controller act. Chapter: [02-06 kube-controller-manager](../02-core-concepts/06-kube-controller-manager.md).

## Where its options are

```bash
kubectl -n kube-system get pods | grep controller-manager           # kubeadm: static Pod
cat /etc/kubernetes/manifests/kube-controller-manager.yaml
cat /etc/systemd/system/kube-controller-manager.service             # from scratch
systemctl status kube-controller-manager
ps -aux | grep [k]ube-controller-manager | tr ' ' '\n' | grep -- '^--'   # either
```

## Is it working?

```bash
kubectl -n kube-system logs kube-controller-manager-controlplane
crictl ps -a | grep kube-controller-manager      # if the Pod is crash-looping
crictl logs <container-id>
kubectl -n kube-system get lease kube-controller-manager -o jsonpath='{.spec.holderIdentity}{"\n"}'

# Quick functional test: does a Deployment produce Pods?
kubectl create deployment cm-test --image=nginx --replicas=2
kubectl get rs,pods -l app=cm-test               # no ReplicaSet/Pods = controller manager problem
kubectl delete deployment cm-test
```

## Fixing it (kubeadm)

```bash
cp /etc/kubernetes/manifests/kube-controller-manager.yaml /root/kcm.yaml.bak   # backup OUTSIDE manifests/
vi /etc/kubernetes/manifests/kube-controller-manager.yaml   # fix the binary name, flag, kubeconfig or cert path
watch crictl ps                                  # the kubelet recreates the Pod
```

## Watching the node controller

```bash
kubectl get nodes -w                             # NotReady ~50 s after a kubelet stops
kubectl describe node node01 | grep -A3 Taints   # node.kubernetes.io/unreachable:NoExecute
kubectl get pod <pod> -o jsonpath='{range .spec.tolerations[*]}{.key}{" "}{.tolerationSeconds}{"\n"}{end}'
                                                 # not-ready / unreachable 300
kubectl get pods -o wide -w                      # Pods evicted after 300 s, replaced on other nodes
```

## Installing the binary

```bash
V=$(curl -sL https://dl.k8s.io/release/stable.txt)
wget https://dl.k8s.io/release/${V}/bin/linux/amd64/kube-controller-manager
```
