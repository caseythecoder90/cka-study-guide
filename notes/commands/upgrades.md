# Version and upgrade commands

Chapters: [06-02 Kubernetes releases](../06-cluster-maintenance/02-kubernetes-releases.md), [06-03 Cluster upgrade process](../06-cluster-maintenance/03-cluster-upgrade-process.md).

## Find versions

```bash
k version                          # kubectl (Client) and kube-apiserver (Server)
k get nodes                        # each node's kubelet version
kubeadm version -o short
kubelet --version                  # on the node
k -n kube-system get pods -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image   # etcd, CoreDNS tags
k api-versions | grep autoscaling  # API versions served
```

## Switch the package repository (every node)

```bash
sudo sed -i 's|/v1.36/|/v1.37/|' /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
apt-cache madison kubeadm                    # available versions, e.g. 1.37.3-1.1
```

## First control plane node

```bash
sudo apt-mark unhold kubeadm && sudo apt-get install -y kubeadm='1.37.3-*' && sudo apt-mark hold kubeadm
kubeadm version -o short
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.37.3           # other control plane nodes: sudo kubeadm upgrade node
k drain controlplane --ignore-daemonsets
sudo apt-mark unhold kubelet kubectl && sudo apt-get install -y kubelet='1.37.3-*' kubectl='1.37.3-*' && sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload && sudo systemctl restart kubelet
k uncordon controlplane
```

## Worker node

```bash
# on node01
sudo apt-mark unhold kubeadm && sudo apt-get install -y kubeadm='1.37.3-*' && sudo apt-mark hold kubeadm
sudo kubeadm upgrade node                    # updates the kubelet configuration
# on the control plane
k drain node01 --ignore-daemonsets
# on node01
sudo apt-mark unhold kubelet kubectl && sudo apt-get install -y kubelet='1.37.3-*' kubectl='1.37.3-*' && sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload && sudo systemctl restart kubelet
# on the control plane
k uncordon node01
k get nodes                                  # VERSION changes after the kubelet restart
```
