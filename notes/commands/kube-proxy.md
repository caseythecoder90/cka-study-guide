# kube-proxy

Checking, configuring and debugging kube-proxy. Chapters: [02-09 kube-proxy](../02-core-concepts/09-kube-proxy.md) · iptables chain walk in [02-01](../02-core-concepts/01-cluster-architecture.md#33-kube-proxy--the-radio-and-routing-charts).

## Is it running everywhere?

```bash
kubectl get nodes
kubectl -n kube-system get daemonset kube-proxy                 # DESIRED / READY = node count
kubectl -n kube-system get pods -o wide | grep kube-proxy       # which node each Pod is on
kubectl -n kube-system logs <kube-proxy-pod>                    # the one on the problem node
kubectl -n kube-system describe daemonset kube-proxy
```

## Its configuration (kubeadm)

```bash
kubectl -n kube-system get configmap kube-proxy -o yaml         # config.conf and kubeconfig.conf
kubectl -n kube-system get configmap kube-proxy -o jsonpath='{.data.config\.conf}' | grep -E 'mode|clusterCIDR'
kubectl -n kube-system get ds kube-proxy -o yaml | grep -A3 command   # --config=/var/lib/kube-proxy/config.conf
kubectl -n kube-system edit configmap kube-proxy
kubectl -n kube-system rollout restart daemonset kube-proxy
kubectl -n kube-system rollout status daemonset kube-proxy
```

## From scratch (systemd)

```bash
systemctl status kube-proxy
cat /etc/systemd/system/kube-proxy.service
cat /var/lib/kube-proxy/kube-proxy-config.yaml
journalctl -u kube-proxy --no-pager | tail -30
```

## Is it doing its job? (on a node)

```bash
curl -s http://localhost:10249/proxyMode                        # iptables / ipvs / nftables
curl -s http://localhost:10256/healthz
kubectl get svc db -o wide                                      # the ClusterIP
kubectl get endpointslices -l kubernetes.io/service-name=db     # its backends
iptables -t nat -L KUBE-SERVICES -n | grep db                   # Service → KUBE-SVC-<hash>
iptables-save -t nat | grep db                                  # every rule for it
kubectl run tmp --image=busybox --rm -it --restart=Never -- wget -qO- -T 3 db   # test from a Pod
```

## Installing the binary

```bash
V=$(curl -sL https://dl.k8s.io/release/stable.txt)
wget https://dl.k8s.io/release/${V}/bin/linux/amd64/kube-proxy
```
