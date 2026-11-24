# CKAD crossover map

> For readers who already hold the CKAD: every CKA curriculum competency, mapped to the related [ckad-exam-prep](https://github.com/caseythecoder90/ckad-exam-prep) chapters and flagged by how much is new. **Review** means you have seen the core of it and can move faster (the text after the dash is what the CKA adds); **New** means budget full study time.
>
> This sets **study pace only**. Every topic still has a complete chapter in this repo — you do not need the CKAD repo to use it. The links here are also the source for each chapter's *Related notes* section.
>
> Competencies are from the CKA curriculum in effect since February 2025 ([cncf/curriculum](https://github.com/cncf/curriculum)). Fill the "CKA chapter" column as chapters are written.

## Cluster Architecture, Installation & Configuration — 25%

| Competency | Related CKAD chapters | Status | CKA chapter |
|---|---|---|---|
| Manage role based access control (RBAC) | [RBAC](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/06-rbac.md) · [Cluster roles](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/07-cluster-roles.md) · [Authorization](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/05-authorization.md) | Review — add user certificates and CSRs | |
| Prepare underlying infrastructure for a cluster | — | **New** — container runtime, kernel modules, sysctl, swap | |
| Create and manage clusters with kubeadm | — | **New** | |
| Manage the cluster lifecycle (upgrades, etcd backup/restore) | — | **New** | [02-03](02-core-concepts/03-etcd-for-beginners.md) (etcd basics) |
| Highly-available control plane | [Architecture](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/01-architecture.md) (single node only) | **New** — stacked vs external etcd, load balancer in front of API servers | [02-03](02-core-concepts/03-etcd-for-beginners.md) (etcd quorum) · [02-04](02-core-concepts/04-etcd-in-kubernetes.md) (topologies) |
| Use Helm and Kustomize to install cluster components | [Helm](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/10-helm-fund/03-helm-exam-patterns.md) · [Kustomize](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/11-kustomize/12-exam-patterns.md) | Review | |
| Extension interfaces (CNI, CSI, CRI) | [Container runtimes](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/02-container-runtimes.md) · [Volume driver plugins](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/08-state-persistance/02-volume-driver-plugins.md) | Partly new — CNI is new | [02-01](02-core-concepts/01-cluster-architecture.md) · [02-02](02-core-concepts/02-docker-vs-containerd.md) (CRI) |
| CRDs, install and configure operators | [CRDs](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/12-custom-resource-definitions.md) · [Custom controllers](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/13-custom-controllers.md) · [Operator framework](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/14-operator-framework.md) | Review | |

## Workloads & Scheduling — 15%

| Competency | Related CKAD chapters | Status | CKA chapter |
|---|---|---|---|
| Deployments, rolling updates and rollbacks | [Deployments](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/06-deployments.md) · [Updates and rollbacks](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/05-pod-design/02-deployment-updates-rollbacks.md) | Review | [02-13](02-core-concepts/13-deployments.md) · [05-01](05-application-lifecycle-management/01-rolling-updates-and-rollbacks.md) (update and rollback cycle) |
| ConfigMaps and Secrets | [ConfigMaps](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/04-configmaps.md) · [Secrets](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/05-secrets.md) | Review — add encryption at rest | [05-04](05-application-lifecycle-management/04-environment-variables.md) (as environment variables) · [05-05](05-application-lifecycle-management/05-configmaps.md) (ConfigMaps, files and volumes) · [05-06](05-application-lifecycle-management/06-secrets.md) (Secrets) · [05-07](05-application-lifecycle-management/07-encrypting-secrets-at-rest.md) (encryption at rest) |
| Workload autoscaling | [Metrics server](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/04-observability/05-monitoring-metrics-server.md) | Partly new — HPA and VPA | |
| Primitives for robust, self-healing deployments | [ReplicaSets](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/05-replicasets.md) · [Probes](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/04-observability/02-liveness-probes.md) | Review — add DaemonSets | [02-06](02-core-concepts/06-kube-controller-manager.md) (controllers) · [02-10](02-core-concepts/10-pods.md) (Pods) · [02-11](02-core-concepts/11-pods-with-yaml.md) (manifests) · [02-12](02-core-concepts/12-replicasets.md) (ReplicaSets) · [03-08](03-scheduling/08-daemonsets.md) (DaemonSets) · [05-08](05-application-lifecycle-management/08-multi-container-pods.md) (multi-container Pods) · [05-09](05-application-lifecycle-management/09-multi-container-design-patterns.md) (init containers, sidecars) |
| Pod admission and scheduling (limits, affinity, ...) | [Resources](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/08-resource-requirements.md) · [Taints](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/10-taints-and-tolerations.md) · [Node selectors](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/11-node-selectors.md) · [Node affinity](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/12-node-affinity.md) · [Admission controllers](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/08-admission-controllers.md) | Review — add static Pods, manual scheduling, multiple schedulers, PriorityClasses, LimitRange and ResourceQuota | [02-07](02-core-concepts/07-kube-scheduler.md) (scheduler basics) · [03-01](03-scheduling/01-manual-scheduling.md) (manual scheduling) · [03-03](03-scheduling/03-taints-and-tolerations.md) (taints) · [03-04](03-scheduling/04-node-selectors.md) (node selectors) · [03-05](03-scheduling/05-node-affinity.md) (node affinity) · [03-06](03-scheduling/06-node-affinity-vs-taints-and-tolerations.md) (dedicated nodes) · [03-07](03-scheduling/07-resource-requirements.md) (requests, limits, LimitRange, ResourceQuota) · [03-09](03-scheduling/09-static-pods.md) (static Pods) · [03-10](03-scheduling/10-priority-classes.md) (PriorityClasses) · [03-11](03-scheduling/11-multiple-schedulers.md) (multiple schedulers) · [03-12](03-scheduling/12-scheduler-profiles.md) (scheduler profiles) · [03-13](03-scheduling/13-admission-controllers.md) (admission controllers) · [03-14](03-scheduling/14-admission-webhooks.md) (admission webhooks) · [02-17](02-core-concepts/17-namespaces.md) (ResourceQuota) |

## Services & Networking — 20%

| Competency | Related CKAD chapters | Status | CKA chapter |
|---|---|---|---|
| Connectivity between Pods | — | **New** — Linux namespaces, bridges, CNI, Pod networking model | |
| Network Policies | [Network policies](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/07-services-and-networking/01-network-policies.md) · [Selectors and rules](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/07-services-and-networking/02-network-policy-selectors-and-rules.md) | Review | |
| ClusterIP, NodePort, LoadBalancer and endpoints | [NodePort](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/08-services-nodeport.md) · [ClusterIP](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/09-services-clusterip.md) | Review — add kube-proxy modes and Service CIDR | [02-09](02-core-concepts/09-kube-proxy.md) (kube-proxy) · [02-14](02-core-concepts/14-services.md) (NodePort, EndpointSlices, ExternalName) · [02-15](02-core-concepts/15-services-clusterip.md) (ClusterIP) · [02-16](02-core-concepts/16-services-loadbalancer.md) (LoadBalancer) |
| Gateway API for ingress traffic | — (KCNA has the theory) | **New** hands-on | |
| Ingress controllers and resources | [Ingress](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/07-services-and-networking/03-ingress.md) | Review — add installing a controller | |
| CoreDNS | — | **New** | [02-15](02-core-concepts/15-services-clusterip.md) (Service names) · [02-17](02-core-concepts/17-namespaces.md) (names across namespaces) |

## Storage — 10%

| Competency | Related CKAD chapters | Status | CKA chapter |
|---|---|---|---|
| StorageClasses and dynamic provisioning | [Storage classes](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/08-state-persistance/05-storage-classes.md) | Review | |
| Volume types, access modes, reclaim policies | [Volumes](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/08-state-persistance/03-kubernetes-volumes.md) · [PV and PVC](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/08-state-persistance/04-persistent-volumes-and-claims.md) | Review | |
| Manage PVs and PVCs | [PV/PVC speed run](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/08-state-persistance/08-pv-pvc-speed-run.md) | Review | |

## Troubleshooting — 30%

| Competency | Related CKAD chapters | Status | CKA chapter |
|---|---|---|---|
| Troubleshoot clusters and nodes | — | **New** — NotReady nodes, kubelet, container runtime | [02-01](02-core-concepts/01-cluster-architecture.md) (kubelet) · [02-06](02-core-concepts/06-kube-controller-manager.md) (node failure) · [02-08](02-core-concepts/08-kubelet.md) (kubelet config) |
| Troubleshoot cluster components | — | **New** — control-plane static Pods, manifests, certificates | [02-01](02-core-concepts/01-cluster-architecture.md) (where each runs) · [02-04](02-core-concepts/04-etcd-in-kubernetes.md) (API server to etcd) · [02-05](02-core-concepts/05-kube-apiserver.md) (API server) |
| Monitor cluster and application resource usage | [Metrics server](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/04-observability/05-monitoring-metrics-server.md) | Review | [04-01](04-logging-and-monitoring/01-monitoring-cluster-components.md) (metrics-server, `kubectl top`) |
| Manage and evaluate container output streams | [Logging](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/04-observability/04-logging.md) | Review — add `crictl` and node-level logs | [04-02](04-logging-and-monitoring/02-managing-application-logs.md) (`kubectl logs`, node log files, `crictl logs`) |
| Troubleshoot services and networking | — | **New** — endpoints, kube-proxy, CoreDNS, CNI | [02-09](02-core-concepts/09-kube-proxy.md) (kube-proxy) |

## Where the time goes

Roughly **45% of the exam weight is new material** (most of Cluster Architecture, all of cluster/node troubleshooting, Pod networking, CoreDNS, Gateway API). The remaining 55% is CKAD review with an administrator's twist. A CKAD holder can spend the course hours accordingly — see [`study-plan.md`](study-plan.md).
