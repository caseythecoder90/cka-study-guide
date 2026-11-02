# kube-apiserver

Finding, reading and fixing the API server's configuration. Chapter: [02-05 kube-apiserver](../02-core-concepts/05-kube-apiserver.md).

## Talking to it

```bash
kubectl get nodes -v=6                           # request URL and status code
kubectl get nodes -v=8                           # plus request and response bodies
kubectl get --raw /api/v1/nodes                  # raw REST call through kubectl's credentials
kubectl get --raw='/readyz?verbose'              # its health checks
kubectl api-resources                            # every resource it serves
```

## Where its options are

```bash
# kubeadm: static Pod
kubectl -n kube-system get pods | grep kube-apiserver
cat /etc/kubernetes/manifests/kube-apiserver.yaml

# From scratch: systemd service
cat /etc/systemd/system/kube-apiserver.service
systemctl status kube-apiserver
journalctl -u kube-apiserver --no-pager | tail -30

# Either: the running process, one flag per line
ps -aux | grep [k]ube-apiserver | tr ' ' '\n' | grep -- '^--'
ps -aux | grep [k]ube-apiserver | tr ' ' '\n' | grep -- '--service-cluster-ip-range'
```

## When it is down (kubectl: connection refused on 6443)

```bash
ssh controlplane
crictl ps -a | grep kube-apiserver               # exited / restarting container
crictl logs <container-id>                       # the error: bad flag, bad path, cannot reach etcd
cp /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.bak   # backup OUTSIDE manifests/
vi /etc/kubernetes/manifests/kube-apiserver.yaml # fix; the kubelet recreates the Pod
watch crictl ps                                  # wait for it to come back
journalctl -u kubelet --no-pager | tail -30      # if the manifest itself is invalid YAML
```

## Installing the binary

```bash
V=$(curl -sL https://dl.k8s.io/release/stable.txt)
wget https://dl.k8s.io/release/${V}/bin/linux/amd64/kube-apiserver
```
