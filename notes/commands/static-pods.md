# Static Pods

Chapter: [03-09 Static Pods](../03-scheduling/09-static-pods.md).

## Find the manifest directory (on the node)

```bash
ssh node01
ps -aux | grep [k]ubelet | grep -o -- '--pod-manifest-path=[^ ]*\|--config=[^ ]*'
grep staticPodPath /var/lib/kubelet/config.yaml           # kubeadm config file
ls -la /etc/kubernetes/manifests/                          # or whatever staticPodPath says
```

## Spot static Pods (from kubectl)

```bash
kubectl get pods -A -o wide                                # name ends in -<node-name>
kubectl get pod static-web-node01 -o jsonpath='{.metadata.ownerReferences[0].kind}{"\n"}'   # Node
kubectl get pods -A -o custom-columns=NS:.metadata.namespace,NAME:.metadata.name,OWNER:.metadata.ownerReferences[0].kind | grep Node
kubectl describe pod static-web-node01 | grep 'Controlled By'                                # Node/node01
```

## Create

```bash
# on the control plane node (kubectl works there)
kubectl run static-busybox --image=busybox --dry-run=client -o yaml \
  --command -- sleep 1000 > static-busybox.yaml
scp static-busybox.yaml node01:/tmp/
ssh node01 'sudo mv /tmp/static-busybox.yaml /etc/kubernetes/manifests/'
kubectl get pod static-busybox-node01 -o wide

# a static Pod on the control plane node itself
kubectl run static-busybox --image=busybox --dry-run=client -o yaml \
  --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```

## Change or delete

```bash
kubectl get pod static-greenbox-node01 -o wide             # which node
ssh node01
grep -l 'name: static-greenbox' /etc/kubernetes/manifests/*   # file name may differ from Pod name
vi /etc/kubernetes/manifests/greenbox.yaml                 # edit: kubelet recreates the Pod
rm /etc/kubernetes/manifests/greenbox.yaml                 # delete: kubelet removes the Pod
cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/kube-apiserver.yaml.bak   # backups OUTSIDE the directory
```

## Change the directory

```bash
vi /var/lib/kubelet/config.yaml                            # staticPodPath: /etc/kubernetes/manifests
systemctl restart kubelet
```

## When the API server is down

```bash
crictl ps -a | grep -E 'kube-apiserver|etcd'               # Exited? ATTEMPT count
crictl logs <container-id>
journalctl -u kubelet --no-pager | tail -30
```

## The Pod did not come back after an edit

```bash
journalctl -u kubelet | grep static-busybox | tail          # "couldn't parse as pod(yaml: line N ...)"
cat -A /etc/kubernetes/manifests/static-busybox.yaml        # tabs show as ^I
ls -la /etc/kubernetes/manifests/                           # delete vim backups ending in ~
kubectl run static-busybox --image=busybox:1.28.4 --dry-run=client -o yaml \
  --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml   # or regenerate it
```
