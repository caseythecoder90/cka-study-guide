# kubectl apply

Declarative updates, the last-applied-configuration annotation, previewing changes, and server-side apply. Chapter: [02-20 kubectl apply](../02-core-concepts/20-kubectl-apply.md).

## Apply and preview

```bash
kubectl apply -f web.yaml                       # create or update
kubectl apply -f ./manifests/                   # every file in a directory
kubectl apply -f ./manifests/ -R                # and subdirectories
kubectl diff -f web.yaml                        # what apply would change (0 = none, 1 = changes)
kubectl apply -f web.yaml --dry-run=server      # validated by the API server, not saved
kubectl apply -f web.yaml --dry-run=client -o yaml
```

## The last-applied-configuration annotation

```bash
kubectl apply view-last-applied deployment/web            # as YAML
kubectl apply view-last-applied -f web.yaml -o json
kubectl get deploy web \
  -o jsonpath='{.metadata.annotations.kubectl\.kubernetes\.io/last-applied-configuration}'

kubectl apply set-last-applied -f web.yaml                # annotation := file, object unchanged
kubectl apply set-last-applied -f web.yaml --create-annotation=true
kubectl apply edit-last-applied deployment/web            # edit the annotation in an editor
```

Write the annotation from commands that do not do it by default:

```bash
kubectl create -f web.yaml --save-config
kubectl replace -f web.yaml --save-config
kubectl edit deployment web --save-config
```

## Hand a field over to kubectl scale or an autoscaler

```bash
# 1. Delete replicas from web.yaml
kubectl apply set-last-applied -f web.yaml      # 2. drop it from the annotation too
kubectl apply -f web.yaml                       # replicas is now left alone
kubectl get deploy web                          # verify the replica count is unchanged
```

## Server-side apply

```bash
kubectl apply --server-side -f web.yaml
kubectl apply --server-side --force-conflicts -f web.yaml     # take ownership on conflict
kubectl apply --server-side --field-manager=my-team -f web.yaml
kubectl get deploy web -o yaml --show-managed-fields          # who owns which field
```

Use `--server-side` when client-side apply fails with `metadata.annotations: Too long`.
