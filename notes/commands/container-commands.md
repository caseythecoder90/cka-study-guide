# Container command and argument commands

Chapters: [05-02 Commands and arguments in Docker](../05-application-lifecycle-management/02-commands-and-arguments-docker.md) · [05-03 Commands and arguments in Kubernetes](../05-application-lifecycle-management/03-commands-and-arguments-kubernetes.md)

## Docker

```bash
docker run ubuntu                        # CMD bash, no terminal: exits at once
docker run -it ubuntu                    # bash with a terminal: stays up
docker run ubuntu sleep 5                # args replace CMD
docker run --entrypoint sleep2.0 img 10  # replaces ENTRYPOINT (and drops the image CMD)
docker ps -a                             # Exited (0) containers
docker image inspect img --format 'EP={{json .Config.Entrypoint}} CMD={{json .Config.Cmd}}'
```

## On a Kubernetes node

```bash
crictl inspecti <image> | grep -A3 -E '"(Entrypoint|Cmd)"'
```

## Kubernetes (command = ENTRYPOINT, args = CMD)

```bash
k run sleeper --image=busybox $do -- sleep 3600              # args
k run sleeper --image=busybox $do --command -- sleep 3600    # command
k get pod sleeper -o jsonpath='{.spec.containers[0].command} {.spec.containers[0].args}'
k edit pod sleeper                                           # refused; edit saved to /tmp/kubectl-edit-*.yaml
k replace --force -f /tmp/kubectl-edit-xxxx.yaml
kubectl explain pod.spec.containers.args
```
