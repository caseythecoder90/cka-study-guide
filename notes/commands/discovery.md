# Discovery: api-resources, explain, help

Finding resources, fields and flags without leaving the terminal. Chapter: [02-19 kubectl explain](../02-core-concepts/19-kubectl-explain.md).

## What does the cluster serve?

```bash
kubectl api-resources                            # NAME, SHORTNAMES, APIVERSION, NAMESPACED, KIND
kubectl api-resources | grep -i <kind>
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
kubectl api-resources --api-group=apps
kubectl api-resources --verbs=list
kubectl api-resources -o wide                    # + VERBS, CATEGORIES
kubectl api-versions                             # group/version pairs
```

## What fields does an object have?

```bash
kubectl explain pods
kubectl explain pods.spec
kubectl explain pods.spec.containers
kubectl explain pods.spec.containers.livenessProbe.httpGet
kubectl explain deploy.spec.strategy.rollingUpdate
kubectl explain deployments --api-version=apps/v1
```

## Where does a field live?

```bash
kubectl explain pods --recursive | less
kubectl explain pods --recursive | grep -i toleration
kubectl explain pods.spec.containers --recursive | grep -A5 securityContext
kubectl explain pods --recursive --max-depth=2
```

## How do I use a command?

```bash
kubectl create --help                            # what can be generated
kubectl create deployment --help
kubectl create service nodeport --help
kubectl run --help
kubectl rollout undo --help
kubectl options                                  # global flags
```
