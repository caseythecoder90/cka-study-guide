# 02 — Managing Application Logs

A containerized app logs by writing to **stdout and stderr**. With Docker, `docker run -d` hides that output and `docker logs -f <id>` shows it. Kubernetes works the same way: `kubectl logs -f <pod>`.

---

## 1. Where the logs live

![The container writes to stdout and stderr; containerd writes it to a file under /var/log/pods, symlinked from /var/log/containers and rotated by the kubelet; the kubelet reads the file for kubectl logs through the API server, and a log-agent DaemonSet tails the same files to ship them off the cluster](./diagrams/02-container-log-path.svg)

- The container runtime writes each container's output to a file on **its node**: `/var/log/pods/<namespace>_<pod>_<uid>/<container>/0.log`. Symlinks to these files sit in `/var/log/containers/`.
- The kubelet rotates the files at `containerLogMaxSize` (default **10Mi**) and keeps `containerLogMaxFiles` (default **5**). Both are set in `/var/lib/kubelet/config.yaml`.
- `kubectl logs` goes through the API server to that node's kubelet, which reads the file. It covers the **current** container and, with `--previous`, the one instance before it. The logs are deleted with the Pod.
- For longer retention, a **log agent** runs as a DaemonSet (Fluent Bit, an OpenTelemetry Collector or Splunk's collector). It reads `/var/log/containers/` on every node and sends the logs to a store outside the cluster.

## 2. kubectl logs

```bash
kubectl logs <pod>                         # one-container Pod
kubectl logs -f <pod>                      # follow (stream)
kubectl logs <pod> -c <container>          # multi-container Pod
kubectl logs <pod> --all-containers
kubectl logs <pod> --previous              # the crashed instance (CrashLoopBackOff)
kubectl logs <pod> --tail=50 --since=1h    # last 50 lines, from the last hour
kubectl logs <pod> --timestamps
kubectl logs deploy/<name>                 # picks one Pod of the Deployment
kubectl logs -l app=web --prefix           # every matching Pod, lines tagged with Pod/container
kubectl logs <pod> -n <ns> > /opt/answer.log   # typical exam output
```

**Slide correction (multi-container Pods):** the lecture says `kubectl logs <pod>` fails and asks for a container name. Current kubectl doesn't fail. It uses the container named in the Pod's `kubectl.kubernetes.io/default-container` annotation, or else the **first** container, and prints `Defaulted container "event-simulator" out of: event-simulator, image-processor`. You can still get the wrong container's logs this way, so pass `-c` whenever there is more than one. List the container names with:

```bash
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].name}'
```

The slide's form `kubectl logs -f event-simulator-pod event-simulator` (container as the second argument) still works. `-c` is clearer.

## 3. When kubectl can't reach the logs

If the API server or the kubelet is down, `kubectl logs` fails. Read the logs on the node instead:

```bash
crictl ps -a --name <container>            # find the container ID, including exited ones
crictl logs <container-id>
ls /var/log/pods/ ; tail /var/log/pods/<ns>_<pod>_<uid>/<container>/0.log
journalctl -u kubelet                      # the kubelet itself is a systemd service, not a container
```

This is how you read the API server's own logs after a bad manifest edit ([03-13](../03-scheduling/13-admission-controllers.md)).

---

## Exam angle

- **"Find the error in Pod X's logs and write it to a file":**
  - `kubectl logs X -n <ns> | grep -i error > /path`
  - Check the namespace.
  - For a multi-container Pod, add `-c`.
- **CrashLoopBackOff:** the current container may have logged nothing yet. Use `--previous` to see why the last one died.
- **API server down:** read the logs with `crictl logs` on the node, or straight from `/var/log/pods/`.
- **Trap:** without `-c`, a multi-container Pod silently gives you the first container's logs. Check the `Defaulted container` line.

## References

- [Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/) — "Pod and container logs" (`kubectl logs` and `--previous`), "Log rotation" (the kubelet settings), "Cluster-level logging architectures" (node agent and sidecar patterns)
- [kubectl logs](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/) — every flag, with examples
- [Debugging Kubernetes nodes with crictl](https://kubernetes.io/docs/tasks/debug/debug-cluster/crictl/) — `crictl ps` and `crictl logs` on a node

## Related notes

- CKAD: [Logging](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/04-observability/04-logging.md) — the developer's view
