# Secret commands

Chapter: [05-06 Secrets](../05-application-lifecycle-management/06-secrets.md). Using them as env: [`env-vars.md`](env-vars.md). The `--from-*` flags match [`configmaps.md`](configmaps.md). Example: [`examples/secrets/`](../../examples/secrets/).

## Create

```bash
k create secret generic app-secret --from-literal=DB_User=root --from-literal=DB_Password=paswrd
k create secret generic app-secret --from-file=app_secret.properties    # one key = whole file
k create secret generic app-secret --from-env-file=app.env              # one key per line
k create secret tls web-tls --cert=tls.crt --key=tls.key
k create secret docker-registry regcred --docker-server=<reg> --docker-username=<u> --docker-password=<p>
k create secret generic app-secret --from-literal=K=V $do > secret.yaml
```

## Encode and decode by hand

```bash
echo -n 'paswrd' | base64          # -n: no trailing newline in the value
base64 -w0 < tls.crt               # no line wrapping for long values
echo -n 'cGFzd3Jk' | base64 -d
```

## Inspect

```bash
k get secrets
k describe secret app-secret                                             # sizes only
k get secret app-secret -o jsonpath='{.data.DB_Password}' | base64 -d    # one value, decoded
k exec <pod> -- cat /opt/app-secret-volumes/DB_Password
```

## Use and change

```bash
k set env deploy/<d> --from=secret/app-secret [--keys=DB_Password]
k create secret generic app-secret --from-literal=K=V2 $do | k apply -f -   # update in place
k rollout restart deploy/<d>                                                # env consumers see changes only after a restart
kubectl explain pod.spec.volumes.secret                                     # secretName, items, defaultMode
```
