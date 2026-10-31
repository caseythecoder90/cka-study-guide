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
| Highly-available control plane | [Architecture](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/01-architecture.md) (single node only) | **New** — stacked vs external etcd, load balancer in front of API servers | [02-03](02-core-concepts/03-etcd-for-beginners.md) (etcd quorum) |
| Use Helm and Kustomize to install cluster components | [Helm](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/10-helm-fund/03-helm-exam-patterns.md) · [Kustomize](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/11-kustomize/12-exam-patterns.md) | Review | |
| Extension interfaces (CNI, CSI, CRI) | [Container runtimes](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/02-container-runtimes.md) · [Volume driver plugins](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/08-state-persistance/02-volume-driver-plugins.md) | Partly new — CNI is new | [02-01](02-core-concepts/01-cluster-architecture.md) · [02-02](02-core-concepts/02-docker-vs-containerd.md) (CRI) |
| CRDs, install and configure operators | [CRDs](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/12-custom-resource-definitions.md) · [Custom controllers](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/13-custom-controllers.md) · [Operator framework](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/14-operator-framework.md) | Review | |

## Workloads & Scheduling — 15%

| Competency | Related CKAD chapters | Status | CKA chapter |
|---|---|---|---|
| Deployments, rolling updates and rollbacks | [Deployments](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/06-deployments.md) · [Updates and rollbacks](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/05-pod-design/02-deployment-updates-rollbacks.md) | Review | |
| ConfigMaps and Secrets | [ConfigMaps](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/04-configmaps.md) · [Secrets](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/05-secrets.md) | Review — add encryption at rest | |
| Workload autoscaling | [Metrics server](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/04-observability/05-monitoring-metrics-server.md) | Partly new — HPA and VPA | |
| Primitives for robust, self-healing deployments | [ReplicaSets](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/05-replicasets.md) · [Probes](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/04-observability/02-liveness-probes.md) | Review — add DaemonSets | |
| Pod admission and scheduling (limits, affinity, ...) | [Resources](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/08-resource-requirements.md) · [Taints](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/10-taints-and-tolerations.md) · [Node selectors](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/11-node-selectors.md) · [Node affinity](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/12-node-affinity.md) · [Admission controllers](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/08-admission-controllers.md) | Review — add static Pods, manual scheduling, multiple schedulers, PriorityClasses, LimitRange and ResourceQuota | |

## Services & Networking — 20%

| Competency | Related CKAD chapters | Status | CKA chapter |
|---|---|---|---|
| Connectivity between Pods | — | **New** — Linux namespaces, bridges, CNI, Pod networking model | |
| Network Policies | [Network policies](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/07-services-and-networking/01-network-policies.md) · [Selectors and rules](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/07-services-and-networking/02-network-policy-selectors-and-rules.md) | Review | |
| ClusterIP, NodePort, LoadBalancer and endpoints | [NodePort](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/08-services-nodeport.md) · [ClusterIP](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/01-core-concepts/09-services-clusterip.md) | Review — add kube-proxy modes and Service CIDR | |
| Gateway API for ingress traffic | — (KCNA has the theory) | **New** hands-on | |
| Ingress controllers and resources | [Ingress](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/07-services-and-networking/03-ingress.md) | Review — add installing a controller | |
| CoreDNS | — | **New** | |

## Storage — 10%

| Competency | Related CKAD chapters | Status | CKA chapter |
|---|---|---|---|
| StorageClasses and dynamic provisioning | [Storage classes](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/08-state-persistance/05-storage-classes.md) | Review | |
| Volume types, access modes, reclaim policies | [Volumes](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/08-state-persistance/03-kubernetes-volumes.md) · [PV and PVC](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/08-state-persistance/04-persistent-volumes-and-claims.md) | Review | |
| Manage PVs and PVCs | [PV/PVC speed run](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/08-state-persistance/08-pv-pvc-speed-run.md) | Review | |

## Troubleshooting — 30%

| Competency | Related CKAD chapters | Status | CKA chapter |
|---|---|---|---|
| Troubleshoot clusters and nodes | — | **New** — NotReady nodes, kubelet, container runtime | [02-01](02-core-concepts/01-cluster-architecture.md) (kubelet) |
| Troubleshoot cluster components | — | **New** — control-plane static Pods, manifests, certificates | [02-01](02-core-concepts/01-cluster-architecture.md) (where each runs) |
| Monitor cluster and application resource usage | [Metrics server](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/04-observability/05-monitoring-metrics-server.md) | Review | |
| Manage and evaluate container output streams | [Logging](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/04-observability/04-logging.md) | Review — add `crictl` and node-level logs | |
| Troubleshoot services and networking | — | **New** — endpoints, kube-proxy, CoreDNS, CNI | |

## Where the time goes

Roughly **45% of the exam weight is new material** (most of Cluster Architecture, all of cluster/node troubleshooting, Pod networking, CoreDNS, Gateway API). The remaining 55% is CKAD review with an administrator's twist. A CKAD holder can spend the course hours accordingly — see [`study-plan.md`](study-plan.md).
