# kubelet

Checking, reading and fixing the kubelet on a node. Chapter: [02-08 kubelet](../02-core-concepts/08-kubelet.md).

## Is it running?

```bash
kubectl get nodes                                # NotReady = start here
ssh node01
systemctl status kubelet
systemctl is-enabled kubelet
journalctl -u kubelet --no-pager | tail -50
journalctl -u kubelet -f                         # follow while restarting
```

## Where its settings are (kubeadm)

```bash
ps -aux | grep [k]ubelet | tr ' ' '\n' | grep -- '^--'    # the flags it is running with
systemctl cat kubelet                                      # unit + drop-ins, merged
cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
cat /var/lib/kubelet/kubeadm-flags.env
cat /etc/default/kubelet                                   # /etc/sysconfig/kubelet on RHEL
cat /var/lib/kubelet/config.yaml
grep -E 'staticPodPath|cgroupDriver|clusterDNS|containerRuntimeEndpoint' /var/lib/kubelet/config.yaml
grep server /etc/kubernetes/kubelet.conf                   # API server address and port
```

## Fixing it

```bash
vi /var/lib/kubelet/config.yaml                  # or kubelet.conf, or the drop-in
systemctl daemon-reload                          # needed if a unit or drop-in changed
systemctl restart kubelet
systemctl enable --now kubelet                   # if it was stopped/disabled
systemctl status containerd                      # the runtime it depends on
kubectl get nodes -w                             # back to Ready
```

## What it is running

```bash
crictl pods                                      # Pod sandboxes the kubelet created
crictl ps -a
ls /etc/kubernetes/manifests/                    # static Pods it reads
kubectl get --raw /api/v1/nodes/node01/proxy/configz | head -c 600   # live kubelet config via the API server
```

## Installing the binary (from scratch)

```bash
V=$(curl -sL https://dl.k8s.io/release/stable.txt)
wget https://dl.k8s.io/release/${V}/bin/linux/amd64/kubelet
```
