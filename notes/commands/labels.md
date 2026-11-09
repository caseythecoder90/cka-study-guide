# Labels, selectors and annotations

Chapter: [03-02 Labels and selectors](../03-scheduling/02-labels-and-selectors.md).

## Filter by label

```bash
kubectl get pods --selector app=App1
kubectl get pods -l app=App1,function=Front-end      # comma = AND
kubectl get pods -l 'env in (prod,dev)'              # either value
kubectl get pods -l 'env notin (qa)'
kubectl get pods -l 'app!=App1'
kubectl get pods -l 'function'                       # label exists
kubectl get pods -l '!function'                      # label not set
kubectl get all -l env=prod                          # common types only
kubectl get pods -l env=prod --no-headers | wc -l    # count
kubectl get pods -A -l app=App1                      # all namespaces
```

## Show labels

```bash
kubectl get pods --show-labels
kubectl get pods -L app,function                     # as columns
kubectl get nodes --show-labels
kubectl describe svc my-service | grep -i selector   # what a Service selects
kubectl get rs simple-webapp -o jsonpath='{.spec.selector}{"\n"}'
```

## Add, change, remove

```bash
kubectl label pod simple-webapp env=prod
kubectl label pod simple-webapp env=dev --overwrite
kubectl label pod simple-webapp env-
kubectl label pods -l app=App1 tier=web              # many objects at once
kubectl run web --image=nginx -l app=App1,tier=web   # labels at creation
```

## Annotations

```bash
kubectl annotate pod simple-webapp buildversion=1.34
kubectl annotate pod simple-webapp buildversion=1.35 --overwrite
kubectl annotate pod simple-webapp buildversion-
kubectl get pod simple-webapp -o jsonpath='{.metadata.annotations}{"\n"}'
```
