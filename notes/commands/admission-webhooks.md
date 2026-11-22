# Admission webhooks

Chapter: [03-14 Validating and Mutating Admission Webhooks](../03-scheduling/14-admission-webhooks.md).

## Inspect

```bash
kubectl get mutatingwebhookconfigurations,validatingwebhookconfigurations    # cluster-scoped
kubectl get validatingwebhookconfiguration <name> -o yaml                     # rules, clientConfig, failurePolicy
kubectl explain validatingwebhookconfiguration.webhooks
```

## Deploy a webhook

```bash
kubectl create ns webhook-demo
kubectl -n webhook-demo create secret tls webhook-server-tls --cert=server.crt --key=server.key
kubectl apply -f webhook-deployment.yaml -f webhook-service.yaml
kubectl -n webhook-demo get pods,endpoints
kubectl apply -f webhook-configuration.yaml
```

## Verify and debug

```bash
kubectl apply -f pod.yaml && kubectl get pod <pod> -o yaml    # fields added by a mutating webhook
kubectl -n webhook-demo logs deploy/<webhook-deployment>
kubectl delete validatingwebhookconfiguration <name>          # webhook down + failurePolicy: Fail = escape hatch
```
