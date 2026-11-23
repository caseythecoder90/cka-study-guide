# ConfigMap commands

Chapter: [05-05 ConfigMaps](../05-application-lifecycle-management/05-configmaps.md). Using them as env: [`env-vars.md`](env-vars.md). Example: [`examples/configmaps/`](../../examples/configmaps/).

## Create

```bash
k create cm app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
k create cm app-config --from-file=app.properties            # one key = whole file
k create cm app-config --from-file=settings=app.properties   # key renamed
k create cm app-config --from-file=config-dir/               # one key per file
k create cm app-config --from-env-file=app.env               # one key per KEY=value line
k create cm app-config --from-file=app.properties $do > cm.yaml
```

## Inspect

```bash
k get cm
k describe cm app-config
k get cm app-config -o jsonpath='{.data.app\.properties}'    # escape dots in key names
k exec <pod> -- ls -la /etc/config                           # files are symlinks into ..data
k exec <pod> -- cat /etc/config/app.properties
```

## Change

```bash
k edit cm app-config
k create cm app-config --from-file=app.properties $do | k apply -f -   # regenerate in place
k rollout restart deploy/<d>      # env and subPath consumers only see changes after a restart
```

## Recall the volume fields

```bash
kubectl explain pod.spec.volumes.configMap
kubectl explain pod.spec.containers.volumeMounts.subPath
```
