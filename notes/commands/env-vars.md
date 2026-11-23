# Environment variable commands

Chapter: [05-04 Environment variables](../05-application-lifecycle-management/04-environment-variables.md). Examples: [`examples/env-vars/`](../../examples/env-vars/).

## Set

```bash
k run webapp --image=nginx --env=APP_COLOR=pink $do > pod.yaml   # literals only; add valueFrom/envFrom by hand
k set env deploy/webapp APP_COLOR=blue                           # starts a rollout
k set env deploy/webapp --from=configmap/app-config              # every key, as configMapKeyRef entries
k set env deploy/webapp --from=secret/app-secret --keys=DB_PASSWORD
k set env deploy/webapp --from=configmap/db-config --prefix=DB_
k set env deploy/webapp APP_COLOR-                               # remove a variable
```

## Check

```bash
k set env deploy/webapp --list
k exec <pod> -- env | sort
k describe pod <pod>                       # CreateContainerConfigError: names the missing ConfigMap/Secret/key
kubectl explain pod.spec.containers.env.valueFrom
kubectl explain pod.spec.containers.envFrom
```

## After changing a ConfigMap or Secret

```bash
k rollout restart deploy/webapp           # env is read only at container start
```
