# Logging commands

Chapter: [04-02 Managing application logs](../04-logging-and-monitoring/02-managing-application-logs.md)

## kubectl logs

```bash
kubectl logs <pod> [-n ns] [-f]
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].name}'   # container names
kubectl logs <pod> -c <container>          # always pass -c in a multi-container Pod
kubectl logs <pod> --all-containers
kubectl logs <pod> --previous              # last crashed instance
kubectl logs <pod> --tail=50 --since=1h --timestamps
kubectl logs deploy/<name>
kubectl logs -l app=web --prefix
kubectl logs <pod> | grep -i error > /opt/answer.log
```

## On the node (API server or kubelet down)

```bash
crictl ps -a --name <container>
crictl logs <container-id>
ls /var/log/pods/                          # <ns>_<pod>_<uid>/<container>/0.log
ls -l /var/log/containers/                 # symlinks to the files above
journalctl -u kubelet -f
grep -i containerLog /var/lib/kubelet/config.yaml   # rotation: 10Mi x 5 by default
```
