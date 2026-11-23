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
| [`namespaces/`](namespaces/) | [02-17 Namespaces](../notes/02-core-concepts/17-namespaces.md) | The `dev` namespace and a ResourceQuota that caps Pods, CPU and memory in it |
| [`services/`](services/) | [02-14 Services](../notes/02-core-concepts/14-services.md) · [02-15 ClusterIP](../notes/02-core-concepts/15-services-clusterip.md) · [02-16 LoadBalancer](../notes/02-core-concepts/16-services-loadbalancer.md) | A NodePort Service selecting the Pods from the Deployment example, an ExternalName alias with no selector, a ClusterIP Service for a back-end tier, and a LoadBalancer Service |
| [`apply/web.yaml`](apply/web.yaml) | [02-20 kubectl apply](../notes/02-core-concepts/20-kubectl-apply.md) | A Deployment with a step-by-step walkthrough of the last-applied annotation: apply, scale, remove a label, apply again |
| [`scheduling/`](scheduling/) | [03-01 Manual scheduling](../notes/03-scheduling/01-manual-scheduling.md) · [03-03 Taints and tolerations](../notes/03-scheduling/03-taints-and-tolerations.md) · [03-04 Node selectors](../notes/03-scheduling/04-node-selectors.md) · [03-05 Node affinity](../notes/03-scheduling/05-node-affinity.md) · [03-06 Affinity vs taints](../notes/03-scheduling/06-node-affinity-vs-taints-and-tolerations.md) | A Pod pinned with `nodeName`, a Binding (JSON) that assigns an existing Pending Pod to a node, a Pod that tolerates `app=blue:NoSchedule`, a Pod that selects nodes labelled `size=Large`, a Deployment with required and preferred node affinity, and a Pod with both a toleration and affinity for a dedicated node |
| [`resources/`](resources/) | [03-07 Resource requirements](../notes/03-scheduling/07-resource-requirements.md) | A Burstable Pod with requests and limits, and a LimitRange with defaults and min/max for the `dev` namespace |
| [`daemonsets/monitoring-daemon.yaml`](daemonsets/monitoring-daemon.yaml) | [03-08 DaemonSets](../notes/03-scheduling/08-daemonsets.md) | A DaemonSet that also runs on the control-plane node, through a toleration |
| [`static-pods/static-web.yaml`](static-pods/static-web.yaml) | [03-09 Static Pods](../notes/03-scheduling/09-static-pods.md) | A Pod manifest to copy into a node's `staticPodPath`; it appears as the mirror Pod `static-web-<node>` |
| [`priority-classes/priority-classes.yaml`](priority-classes/priority-classes.yaml) | [03-10 PriorityClasses](../notes/03-scheduling/10-priority-classes.md) | A preempting and a non-preempting PriorityClass, and a Pod using `priorityClassName` |
| [`multiple-schedulers/my-scheduler.yaml`](multiple-schedulers/my-scheduler.yaml) | [03-11 Multiple Schedulers](../notes/03-scheduling/11-multiple-schedulers.md) | A second kube-scheduler Pod reading its config from a ConfigMap, and a Pod using `schedulerName` |
| [`scheduler-profiles/scheduler-config.yaml`](scheduler-profiles/scheduler-config.yaml) | [03-12 Scheduler Profiles](../notes/03-scheduling/12-scheduler-profiles.md) | One KubeSchedulerConfiguration with three profiles: default, TaintToleration disabled, scoring disabled |
| [`env-vars/env-sources.yaml`](env-vars/env-sources.yaml) | [05-04 Environment Variables](../notes/05-application-lifecycle-management/04-environment-variables.md) | A ConfigMap, a Secret and a Pod using every source: literal, `configMapKeyRef`, `secretKeyRef`, `optional`, downward API, `$(VAR)`, and `envFrom` with a prefix |
| [`env-vars/deployment-envfrom.yaml`](env-vars/deployment-envfrom.yaml) | [05-04 Environment Variables](../notes/05-application-lifecycle-management/04-environment-variables.md) | A Deployment loading a whole ConfigMap with `envFrom` and one Secret key with `secretKeyRef` |
