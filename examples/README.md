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
