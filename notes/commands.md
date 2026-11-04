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

### 2c. etcd for beginners (chapter 02-03)

Per-topic detail: [`commands/etcd.md`](commands/etcd.md).

```bash
# Install from release binaries (lab only — kubeadm runs etcd as a static Pod)
ETCD_VER=v3.x.y
curl -L https://github.com/etcd-io/etcd/releases/download/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o etcd.tar.gz
tar xzvf etcd.tar.gz && cd etcd-${ETCD_VER}-linux-amd64
./etcd                                           # listens on 2379 (client) and 2380 (peer)

export ETCDCTL_API=3                             # default from etcd v3.4; harmless to set
etcdctl version
etcdctl put key1 value1
etcdctl get key1
etcdctl get key1 -w json                         # create_revision, mod_revision, version
etcdctl get /app --prefix
etcdctl get "" --prefix --keys-only              # every key
etcdctl del key1
etcdctl watch key1                               # stream changes (another terminal: put)
etcdctl watch key1 --rev=1                       # replay from a revision
etcdctl endpoint health
etcdctl member list
etcdutl version                                  # offline tool: restore, status, defrag
```

### 2d. etcd in Kubernetes (chapter 02-04)

Per-topic detail: [`commands/etcd.md`](commands/etcd.md#the-clusters-etcd-kubeadm).

```bash
kubectl -n kube-system get pods | grep etcd      # kubeadm: static Pod etcd-<node>
systemctl status etcd                            # from scratch: systemd service
cat /etc/kubernetes/manifests/etcd.yaml          # data-dir, client URLs, cert paths
grep etcd /etc/kubernetes/manifests/kube-apiserver.yaml   # --etcd-servers, --etcd-cafile/certfile/keyfile

# etcdctl inside the etcd Pod (no sh -c: the image may have no shell)
kubectl -n kube-system exec etcd-controlplane -- etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get / --prefix --keys-only --limit=10
# ... get /registry/pods --prefix --keys-only
# ... get /registry/minions --prefix --keys-only     # nodes
# ... member list -w table
```

### 2e. kube-apiserver (chapter 02-05)

Per-topic detail: [`commands/kube-apiserver.md`](commands/kube-apiserver.md).

```bash
kubectl get nodes -v=6                           # the HTTPS request behind a kubectl command
kubectl get --raw /api/v1/nodes                  # same request, raw JSON
kubectl -n kube-system get pods | grep apiserver # kubeadm: static Pod kube-apiserver-<node>
cat /etc/kubernetes/manifests/kube-apiserver.yaml          # kubeadm
cat /etc/systemd/system/kube-apiserver.service             # from scratch
ps -aux | grep [k]ube-apiserver | tr ' ' '\n' | grep -- '^--'   # either: the flags it actually runs with
crictl ps -a | grep kube-apiserver               # API server down: find the container...
crictl logs <container-id>                       # ...and read why
wget https://dl.k8s.io/release/v1.xx.y/bin/linux/amd64/kube-apiserver
```

### 2f. kube-controller-manager (chapter 02-06)

Per-topic detail: [`commands/kube-controller-manager.md`](commands/kube-controller-manager.md).

```bash
kubectl -n kube-system get pods | grep controller-manager
cat /etc/kubernetes/manifests/kube-controller-manager.yaml      # kubeadm
cat /etc/systemd/system/kube-controller-manager.service         # from scratch
ps -aux | grep [k]ube-controller-manager | tr ' ' '\n' | grep -- '^--'
kubectl -n kube-system logs kube-controller-manager-controlplane
kubectl -n kube-system get lease kube-controller-manager -o jsonpath='{.spec.holderIdentity}{"\n"}'   # active leader
kubectl get nodes                                # NotReady after ~50 s of silence
kubectl describe node node01 | grep -A3 Taints   # node.kubernetes.io/unreachable
kubectl get pod <pod> -o jsonpath='{.spec.tolerations}'   # the default 300 s tolerations
```

### 2g. kube-scheduler (chapter 02-07)

Per-topic detail: [`commands/kube-scheduler.md`](commands/kube-scheduler.md).

```bash
kubectl -n kube-system get pods | grep scheduler
cat /etc/kubernetes/manifests/kube-scheduler.yaml               # kubeadm
cat /etc/systemd/system/kube-scheduler.service                  # from scratch (settings in --config file)
ps -aux | grep [k]ube-scheduler | tr ' ' '\n' | grep -- '^--'
kubectl get pods --field-selector spec.nodeName= -A             # Pods with no node yet
kubectl describe pod <pod> | grep -A5 Events                    # FailedScheduling: which filter failed
kubectl get events -A --field-selector reason=FailedScheduling
kubectl get pod <pod> -o jsonpath='{.spec.nodeName}{"\n"}'      # where it was placed
```

### 2h. kubelet (chapter 02-08)

Per-topic detail: [`commands/kubelet.md`](commands/kubelet.md).

```bash
ssh node01
systemctl status kubelet
systemctl enable --now kubelet                   # if stopped
journalctl -u kubelet --no-pager | tail -50      # why it is failing
ps -aux | grep [k]ubelet | tr ' ' '\n' | grep -- '^--'
systemctl cat kubelet                            # unit + 10-kubeadm.conf drop-in
cat /var/lib/kubelet/config.yaml                 # staticPodPath, cgroupDriver, clusterDNS ...
cat /var/lib/kubelet/kubeadm-flags.env
grep server /etc/kubernetes/kubelet.conf         # API server address it uses
systemctl daemon-reload && systemctl restart kubelet
kubectl get nodes -o wide                        # Ready again?
```

### 2i. kube-proxy (chapter 02-09)

Per-topic detail: [`commands/kube-proxy.md`](commands/kube-proxy.md).

```bash
kubectl -n kube-system get daemonset kube-proxy                 # DESIRED = number of nodes
kubectl -n kube-system get pods -o wide | grep kube-proxy       # one per node
kubectl -n kube-system get configmap kube-proxy -o yaml         # config.conf: mode, clusterCIDR ...
kubectl -n kube-system edit configmap kube-proxy
kubectl -n kube-system rollout restart daemonset kube-proxy     # pick up the change
kubectl -n kube-system logs ds/kube-proxy | head
curl -s http://localhost:10249/proxyMode                        # on a node
iptables -t nat -L KUBE-SERVICES -n | grep <service-name>
```

### 2j. Pods (chapter 02-10)

Per-topic detail: [`commands/pods.md`](commands/pods.md).

```bash
kubectl run nginx --image=nginx                  # one Pod, named nginx
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl run busybox --image=busybox --restart=Never -- sleep 3600
kubectl run tmp --image=busybox --rm -it --restart=Never -- sh
kubectl get pods -o wide                         # Pod IP and node
kubectl describe pod nginx                       # events at the bottom
kubectl get pod nginx -o yaml
kubectl logs nginx -c <container>                # one container of a multi-container Pod
kubectl exec -it nginx -c <container> -- sh
kubectl delete pod nginx
```

### 2k. Pods with YAML (chapter 02-11)

Per-topic detail: [`commands/pods.md`](commands/pods.md#manifests).

```bash
kubectl run myapp-pod --image=nginx --labels=app=myapp,type=front-end --dry-run=client -o yaml > pod.yaml
kubectl apply -f pod.yaml --dry-run=server       # full validation, stores nothing
kubectl apply -f pod.yaml                        # create or update
kubectl create -f pod.yaml                       # errors if it exists
kubectl replace --force -f pod.yaml              # delete and recreate
kubectl api-resources | grep -i deployment       # correct apiVersion for a kind
kubectl explain pod.spec.containers              # fields, types, -required-
kubectl explain pod --recursive | less
```

### 2l. ReplicaSets (chapter 02-12)

Per-topic detail: [`commands/replicasets.md`](commands/replicasets.md).

```bash
kubectl apply -f replicaset-definition.yaml
kubectl get rs -o wide                           # DESIRED, CURRENT, READY, IMAGES, SELECTOR
kubectl get rc                                   # legacy ReplicationControllers
kubectl describe rs myapp-replicaset
kubectl get pods -l type=front-end --show-labels
kubectl scale --replicas=6 rs/myapp-replicaset   # file unchanged
kubectl scale --replicas=6 -f replicaset-definition.yaml
kubectl edit rs myapp-replicaset                 # running Pods keep the old template
kubectl delete pod -l type=front-end             # recreate from the new template
kubectl delete rs myapp-replicaset               # also deletes its Pods
kubectl delete rs myapp-replicaset --cascade=orphan   # keeps the Pods
```

### 2m. Deployments (chapter 02-13)

Per-topic detail: [`commands/deployments.md`](commands/deployments.md).

```bash
kubectl create deployment myapp --image=nginx:1.27 --replicas=3 --dry-run=client -o yaml > deploy.yaml
kubectl apply -f deploy.yaml
kubectl get deploy,rs,pods
kubectl get all
kubectl set image deployment/myapp <container>=nginx:1.28
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp
kubectl rollout history deployment/myapp --revision=2
kubectl rollout undo deployment/myapp
kubectl rollout undo deployment/myapp --to-revision=1
kubectl rollout pause deployment/myapp
kubectl rollout resume deployment/myapp
kubectl rollout restart deployment/myapp
kubectl scale deployment/myapp --replicas=6
kubectl explain deployment.spec.strategy.rollingUpdate
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
