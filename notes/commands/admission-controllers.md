# Admission controllers

Chapter: [03-13 Admission Controllers](../03-scheduling/13-admission-controllers.md).

## Inspect

```bash
grep admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml           # configured on/off
ps -ef | grep [k]ube-apiserver | grep -o -- '--[a-z-]*admission-plugins=[^ ]*'  # running process
kubectl -n kube-system exec kube-apiserver-controlplane -- kube-apiserver -h | grep enable-admission-plugins   # defaults
```

## Enable or disable

```bash
cp /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.bak   # never back up inside manifests/
vi /etc/kubernetes/manifests/kube-apiserver.yaml
#   - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
#   - --disable-admission-plugins=DefaultStorageClass
watch crictl ps --name kube-apiserver          # wait for the new container
kubectl -n kube-system get pod kube-apiserver-controlplane
```

## Verify

```bash
kubectl run nginx --image=nginx -n blue        # NamespaceAutoProvision: creates ns blue
kubectl get ns blue
```

## API server did not come back

```bash
crictl ps -a --name kube-apiserver
crictl logs <container-id>                     # bad flag / unknown plugin name
journalctl -u kubelet | tail -20               # manifest parse errors
```
