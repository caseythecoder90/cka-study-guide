# Examples

Runnable manifests and scripts that accompany [`../notes/`](../notes/), so each task can be reproduced against a practice cluster (kind, a kubeadm cluster on VMs, or the KodeKloud labs).

```bash
kubectl apply -f <file>.yaml
```

## Contents

| Path | Chapter | What it shows |
|---|---|---|
| [`pods/pod-definition.yaml`](pods/pod-definition.yaml) | [02-11 Pods with YAML](../notes/02-core-concepts/11-pods-with-yaml.md) | A single-container Pod with the four top-level fields and two labels |
| [`replicasets/`](replicasets/) | [02-12 ReplicaSets](../notes/02-core-concepts/12-replicasets.md) | A ReplicaSet with `matchLabels` and the legacy ReplicationController equivalent, which has no selector |
| [`deployments/deployment-definition.yaml`](deployments/deployment-definition.yaml) | [02-13 Deployments](../notes/02-core-concepts/13-deployments.md) | A Deployment with the `RollingUpdate` strategy, `maxSurge` and `maxUnavailable` written out at their defaults |
| [`services/`](services/) | [02-14 Services](../notes/02-core-concepts/14-services.md) · [02-15 ClusterIP](../notes/02-core-concepts/15-services-clusterip.md) · [02-16 LoadBalancer](../notes/02-core-concepts/16-services-loadbalancer.md) | A NodePort Service selecting the Pods from the Deployment example, an ExternalName alias with no selector, a ClusterIP Service for a back-end tier, and a LoadBalancer Service |
