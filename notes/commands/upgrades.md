# Version and upgrade commands

Chapters: [06-02 Kubernetes releases](../06-cluster-maintenance/02-kubernetes-releases.md).

## Find versions

```bash
k version                          # kubectl (Client) and kube-apiserver (Server)
k get nodes                        # each node's kubelet version
kubeadm version -o short
kubelet --version                  # on the node
k -n kube-system get pods -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image   # etcd, CoreDNS tags
k api-versions | grep autoscaling  # API versions served
```
