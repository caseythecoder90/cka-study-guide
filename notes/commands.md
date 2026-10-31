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
