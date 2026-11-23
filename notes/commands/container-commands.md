# Container command and argument commands

Chapters: [05-02 Commands and arguments in Docker](../05-application-lifecycle-management/02-commands-and-arguments-docker.md)

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
