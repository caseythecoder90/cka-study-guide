# Multi-container Pod commands

Chapters: [05-08 Multi-container Pods](../05-application-lifecycle-management/08-multi-container-pods.md) · [05-09 Design patterns](../05-application-lifecycle-management/09-multi-container-design-patterns.md). Example: [`examples/multi-container/`](../../examples/multi-container/).

## Create

```bash
k run web --image=nginx $do > pod.yaml      # then add a second entry under spec.containers
k get pod web -o yaml > web.yaml            # add a sidecar to a RUNNING Pod: edit the file...
k replace --force -f web.yaml               # ...then delete and recreate (containers can't be edited live)
k edit deploy web                           # in a Deployment: edit the template, a rollout follows
```

## Inspect

```bash
k get pod web                                         # READY 2/2
k get pod web -o jsonpath='{.spec.containers[*].name}'
k describe pod web                                    # which container is unready / restarting
k logs web -c sidecar [--previous] [-f]
k logs web --all-containers
k logs deploy/web -c sidecar                          # through the Deployment
k exec -it web -c sidecar -- sh
k exec web -c sidecar -- ls /var/log/shared           # check a shared emptyDir
```

## Recall the fields

```bash
kubectl explain pod.spec.containers
kubectl explain pod.spec.initContainers.restartPolicy     # native sidecar: Always
kubectl explain pod.spec.volumes.emptyDir
```

## Init containers and sidecars

```bash
k get pod app                                        # Init:0/2, Init:Error, Init:CrashLoopBackOff, PodInitializing
k describe pod app                                   # "Init Containers:" section
k logs app -c wait-for-db [--previous]               # init containers need -c
k get pod app -o jsonpath='{.spec.initContainers[*].name}'
```

## Which container restarted

```bash
k get pod app -o jsonpath='{range .status.containerStatuses[*]}{.name}{"\t"}{.restartCount}{"\t"}{.ready}{"\n"}{end}'
k get pod app -o jsonpath='{.metadata.uid}'          # unchanged = same Pod
```
