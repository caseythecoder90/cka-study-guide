# Deployments

Creating, updating, rolling back and scaling Deployments. Chapter: [02-13 Deployments](../02-core-concepts/13-deployments.md).

## Create

```bash
kubectl create deployment myapp --image=nginx:1.27 --replicas=3
kubectl create deployment myapp --image=nginx:1.27 --replicas=3 --dry-run=client -o yaml > deploy.yaml
kubectl apply -f deploy.yaml
```

## Inspect

```bash
kubectl get deployments                          # READY, UP-TO-DATE, AVAILABLE
kubectl get deploy,rs,pods
kubectl get all                                  # skips ConfigMaps, Secrets, Ingresses, PVCs and others
kubectl describe deployment myapp                # strategy, conditions, scale events
kubectl get deployment myapp -o jsonpath='{.spec.template.spec.containers[*].name}{"\n"}'
kubectl explain deployment.spec.strategy.rollingUpdate
```

## Update

```bash
kubectl set image deployment/myapp <container>=nginx:1.28
kubectl edit deployment myapp
kubectl apply -f deploy.yaml
kubectl rollout status deployment/myapp          # waits; non-zero exit on failure
kubectl get rs                                   # new ReplicaSet up, old one at 0
kubectl rollout restart deployment/myapp         # new Pods, same configuration
```

## History and rollback

```bash
kubectl rollout history deployment/myapp
kubectl rollout history deployment/myapp --revision=2
kubectl rollout undo deployment/myapp
kubectl rollout undo deployment/myapp --to-revision=1
kubectl annotate deployment/myapp kubernetes.io/change-cause="image updated to 1.28"
```

## Pause and resume

```bash
kubectl rollout pause deployment/myapp
kubectl set image deployment/myapp <container>=nginx:1.28
kubectl rollout resume deployment/myapp
```

## Scale and delete

```bash
kubectl scale deployment/myapp --replicas=6      # no new revision
kubectl delete deployment myapp                  # also deletes its ReplicaSets and Pods
```
