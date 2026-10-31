# Container runtimes

`crictl`, `ctr` and `nerdctl` against containerd. Chapter: [02-02 Docker vs containerd](../02-core-concepts/02-docker-vs-containerd.md).

## crictl — use this on the exam

Talks CRI to any runtime, exactly as the kubelet does, and understands Pods. **Inspect and debug only** — the kubelet removes containers it did not create.

```bash
crictl pods                                      # Pod sandboxes on this node
crictl pods --name kube-apiserver                # filter by Pod name
crictl ps                                        # running containers
crictl ps -a                                     # include exited (crash-looping) ones
crictl ps --pod <pod-id>                         # containers in one Pod
crictl logs <container-id>                       # IDs can be shortened
crictl logs --tail 20 <container-id>
crictl exec -it <container-id> sh
crictl inspect <container-id>                    # container JSON
crictl inspectp <pod-id>                         # Pod sandbox JSON
crictl inspecti <image>                          # image JSON
crictl images
crictl pull busybox
crictl rmi <image>
crictl stats                                     # CPU / memory per container
crictl info                                      # runtime status and config
crictl version
```

### Pointing crictl at the runtime

```bash
cat /etc/crictl.yaml
crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/containerd/containerd.sock
grep containerRuntimeEndpoint /var/lib/kubelet/config.yaml   # what the kubelet uses
```

```yaml
# /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
```

| Runtime | CRI socket |
|---|---|
| containerd | `unix:///run/containerd/containerd.sock` |
| CRI-O | `unix:///var/run/crio/crio.sock` |
| cri-dockerd | `unix:///var/run/cri-dockerd.sock` |

## ctr — ships with containerd

```bash
ctr namespaces ls                                # default, k8s.io, moby ...
ctr -n k8s.io containers ls                      # Kubernetes' containers
ctr -n k8s.io images ls
ctr images pull docker.io/library/redis:alpine   # fully qualified names only
ctr run docker.io/library/redis:alpine redis
```

## nerdctl — Docker-compatible

```bash
nerdctl run --name redis redis:alpine
nerdctl run --name webserver -p 80:80 -d nginx
nerdctl ps
nerdctl --namespace k8s.io ps                    # Kubernetes' containers
nerdctl compose up
```

## The runtime on the node

```bash
systemctl status containerd
journalctl -u containerd --no-pager | tail -30
ps -ef | grep -E 'containerd|runc' | grep -v grep   # containerd + one containerd-shim-runc-v2 per Pod
kubectl get nodes -o wide                        # CONTAINER-RUNTIME column, e.g. containerd://1.7.x
```
