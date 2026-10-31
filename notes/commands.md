# Commands reference

> Every `kubectl`, `kubeadm`, `etcdctl`, `crictl`, `systemctl` and shell command from the course and its labs, collected in one page to `Ctrl+F`. The CKA is hands-on — this is exam material, not supporting material. For focused per-topic files see [`commands/`](commands/).

## How to use this file

- `Ctrl+F` for the tool or resource (`etcdctl snapshot`, `kubeadm upgrade`, `crictl ps`, ...).
- Sections follow the course sections and are added as the course reaches them.

---

## 0. Exam environment

Per-topic detail: [`commands/setup.md`](commands/setup.md).

```bash
alias k >/dev/null 2>&1 || alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
kubectl config use-context <context-from-the-task>   # before every task
```

## 2. Core concepts

### 2a. Cluster architecture (chapter 02-01)

Per-topic detail: [`commands/cluster-architecture.md`](commands/cluster-architecture.md).

```bash
kubectl get nodes -o wide                        # roles, versions, internal IPs, container runtime
kubectl -n kube-system get pods -o wide          # control plane static Pods: <component>-<node-name>
kubectl get pods -v=6                            # shows the HTTPS request kubectl sends to :6443
kubectl get --raw='/readyz?verbose'              # API server health checks, including etcd
kubectl -n kube-node-lease get leases            # one heartbeat Lease per node

# On a control plane node
ls /etc/kubernetes/manifests/                    # etcd, kube-apiserver, kube-controller-manager, kube-scheduler
crictl ps -a | grep kube-apiserver               # when kubectl itself is down
crictl logs <container-id>

# On any node
systemctl status kubelet
journalctl -u kubelet --no-pager | tail -50
systemctl status containerd
cat /var/lib/kubelet/config.yaml

# kube-proxy
kubectl -n kube-system get daemonset kube-proxy
kubectl -n kube-system get configmap kube-proxy -o yaml | grep mode
curl -s http://localhost:10249/proxyMode         # on a node
iptables -t nat -L KUBE-SERVICES -n | grep <service-name>
iptables-save -t nat | grep <service-name>
```

### 2b. Docker vs containerd (chapter 02-02)

Per-topic detail: [`commands/container-runtimes.md`](commands/container-runtimes.md).

```bash
# crictl — the exam tool; talks CRI like the kubelet, knows about Pods
crictl pods
crictl ps -a                                     # exited containers too
crictl ps -a | grep kube-apiserver               # control plane crash-loop: find it...
crictl logs <container-id>                       # ...and read why
crictl exec -it <container-id> sh
crictl inspect <container-id>; crictl inspectp <pod-id>; crictl inspecti <image>
crictl images; crictl pull busybox; crictl rmi <image>
crictl info
crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/containerd/containerd.sock
cat /etc/crictl.yaml                             # runtime-endpoint / image-endpoint

# ctr — ships with containerd, low level, full image names
ctr namespaces ls
ctr -n k8s.io containers ls                      # Kubernetes' containers live in k8s.io
ctr images pull docker.io/library/redis:alpine

# nerdctl — Docker-compatible CLI for containerd
nerdctl run --name webserver -p 80:80 -d nginx
nerdctl --namespace k8s.io ps

# The runtime processes on a node
systemctl status containerd
ps -ef | grep -E 'containerd|runc' | grep -v grep   # daemon + one shim per Pod; runc already exited
grep containerRuntimeEndpoint /var/lib/kubelet/config.yaml
```

## 3. Scheduling

## 4. Logging and monitoring

## 5. Application lifecycle management

## 6. Cluster maintenance

## 7. Security

## 8. Storage

## 9. Networking

## 10. Design and install a cluster

## 11. Install with kubeadm

## 12. Helm

## 13. Kustomize

## 14. Troubleshooting

## 15. Other topics — JSONPath and advanced kubectl
