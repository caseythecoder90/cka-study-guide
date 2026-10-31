# Cluster architecture

Finding each component, checking its health, and reading what it is doing. Chapter: [02-01 Cluster Architecture](../02-core-concepts/01-cluster-architecture.md).

## The cluster from kubectl

```bash
kubectl get nodes -o wide                        # ROLES, VERSION, INTERNAL-IP, CONTAINER-RUNTIME
kubectl -n kube-system get pods -o wide          # control plane Pods are named <component>-<node-name>
kubectl get --raw='/readyz?verbose'              # API server readiness checks, etcd included
kubectl -n kube-node-lease get leases            # kubelet heartbeats, one per node
kubectl get pods -v=6                            # see the REST call: GET https://...:6443/api/v1/namespaces/default/pods
kubectl get pods -v=8                            # also prints request and response bodies
```

## Control plane (static Pods)

```bash
ssh controlplane
ls /etc/kubernetes/manifests/                    # one YAML per component; edit the file to change it
crictl ps -a | grep -E 'kube-apiserver|etcd|kube-scheduler|kube-controller-manager'
crictl logs <container-id>                       # works when the API server is down
kubectl -n kube-system logs kube-apiserver-<node-name>   # works when it is up
```

## kubelet and containerd (systemd services)

```bash
systemctl status kubelet
systemctl restart kubelet
journalctl -u kubelet --no-pager | tail -50
cat /var/lib/kubelet/config.yaml                 # staticPodPath, cgroupDriver, clusterDNS, ...
systemctl status containerd
crictl info | head -20                           # runtime the kubelet is talking to
crictl pods                                      # Pod sandboxes on this node
crictl ps                                        # running containers on this node
```

## kube-proxy (DaemonSet)

```bash
kubectl -n kube-system get daemonset kube-proxy
kubectl -n kube-system logs ds/kube-proxy | head
kubectl -n kube-system get configmap kube-proxy -o yaml | grep mode   # "" means iptables on Linux
curl -s http://localhost:10249/proxyMode         # on a node: iptables, ipvs or nftables

# The rules for one Service
kubectl get svc web -o wide                      # its ClusterIP
kubectl get endpointslices -l kubernetes.io/service-name=web
iptables -t nat -L KUBE-SERVICES -n | grep web   # Service → KUBE-SVC-<hash>
iptables -t nat -L KUBE-SVC-<hash> -n            # one KUBE-SEP per endpoint, with probabilities
iptables -t nat -L KUBE-SEP-<hash> -n            # the DNAT to podIP:targetPort
```
