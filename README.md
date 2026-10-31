# CKA Study Guide

Exam-focused notes for the **Certified Kubernetes Administrator (CKA)** exam, built while working through Mumshad Mannambeth's Udemy course *Certified Kubernetes Administrator (CKA) with Practice Tests* (KodeKloud) and expanded with the commands, file paths, and troubleshooting order that a hands-on exam rewards.

The CKA is performance-based: real clusters, a terminal, and a clock. These notes optimize for speed: imperative generators, where every component's config lives on a node, how each component fails, and how to verify a fix. Every chapter ends with an *Exam angle* section describing what a task on that topic looks like and the trap that costs points.

**This repo is a complete CKA resource on its own** — you do not need anything else from this series to use it. It is the third of five certifications on the road to CNCF [Kubestronaut](https://www.cncf.io/training/kubestronaut/) status, and each certification gets its own self-contained repo: [ckad-exam-prep](https://github.com/caseythecoder90/ckad-exam-prep) (CKAD) and [kcna-study-guide](https://github.com/caseythecoder90/kcna-study-guide) (KCNA) so far. Where a topic also appears in another exam, the chapter covers it fully here and ends with *Related notes* linking the sibling chapter, so Kubestronaut candidates can see the same topic from the developer's or the conceptual angle.

## Who this is for

- Anyone preparing for the CKA who wants dense, task-oriented notes and a single page of node paths and ports to drill.
- CKAD holders moving on to the CKA who want to know exactly which parts are new — [`notes/ckad-crossover.md`](notes/ckad-crossover.md) maps it out.
- People collecting the Kubestronaut set who want one consistent study resource per exam.

## Repository layout

```
cka-study-guide/
├── README.md
├── notes/                         # the study notes, one folder per course section (added as the course goes)
│   ├── study-plan.md              # schedule: Nov-Dec course, January exam, Kubestronaut roadmap
│   ├── ckad-crossover.md          # for CKAD holders: which CKA topics are review vs new
│   ├── exam-facts.md              # recall sheet: node paths, ports, defaults, version skew, etcd flags
│   ├── commands/                  # per-topic command references, including imperative.md
│   ├── commands.md                # global single-file command reference (Ctrl+F)
│   └── NOTES-WORKFLOW.md          # how these notes are authored
├── tools/diagrams/                # shared PlantUML theme + render script for every diagram
└── examples/                      # runnable manifests and scripts from the labs
```

## Contents

| Section | Covers | Status |
|---|---|---|
| [02 · Core Concepts](notes/02-core-concepts/) | Cluster architecture, etcd, kube-apiserver, controller manager, scheduler, kubelet, kube-proxy, Pods, ReplicaSets, Deployments, Services, namespaces | In progress |
| 03 · Scheduling | Manual scheduling, labels and selectors, taints and tolerations, node affinity, resource limits, DaemonSets, static Pods, PriorityClasses, multiple schedulers, scheduler profiles | Not started |
| 04 · Logging & Monitoring | metrics-server, `kubectl top`, application logs | Not started |
| 05 · Application Lifecycle Management | Rolling updates and rollbacks, commands and args, env, ConfigMaps, Secrets, encryption at rest, multi-container and init containers, autoscaling | Not started |
| 06 · Cluster Maintenance | OS upgrades, drain and cordon, version skew, kubeadm cluster upgrade, etcd backup and restore | Not started |
| 07 · Security | TLS and PKI, certificate creation and inspection, Certificates API, kubeconfig, API groups, RBAC, ServiceAccounts, image security, security contexts, NetworkPolicies, CRDs | Not started |
| 08 · Storage | Volumes, PersistentVolumes and claims, StorageClasses, CSI | Not started |
| 09 · Networking | Linux networking, network namespaces, CNI, Pod and Service networking, CoreDNS, Ingress, Gateway API | Not started |
| 10 · Design and Install a Cluster | Cluster design choices, HA control plane, etcd topologies | Not started |
| 11 · Install with kubeadm | Provisioning nodes, container runtime, `kubeadm init` and `join`, installing a CNI | Not started |
| 12 · Helm Basics (2025 update) | Charts, repositories, install/upgrade/rollback | Not started |
| 13 · Kustomize Basics (2025 update) | Bases, overlays, patches, transformers | Not started |
| 14 · Troubleshooting | Application failure, control-plane failure, worker-node failure, network troubleshooting | Not started |
| 15 · Other Topics | JSONPath, custom columns, sorting, advanced kubectl | Not started |

Section numbers match the course sections. Sections 1 (Introduction), 16 (Lightning Labs) and 17 (Mock Exams) have no folder; their lessons feed the recall sheet and command references.

## How to use it

1. **Learn a topic** — read its chapter in `notes/<section>/`: concepts, commands and file paths, then an *Exam angle* section with the trap to avoid. Every chapter is complete without the other repos.
2. **Plan your pace** — [`notes/study-plan.md`](notes/study-plan.md) is a week-by-week schedule you can adapt. Already hold the CKAD? [`notes/ckad-crossover.md`](notes/ckad-crossover.md) shows which topics you can move through faster.
4. **Drill the facts** — [`notes/exam-facts.md`](notes/exam-facts.md) is every path, port, and default you should not have to search for during the exam.
5. **Drill the commands** — [`notes/commands/imperative.md`](notes/commands/imperative.md) for generators, [`notes/commands.md`](notes/commands.md) for everything else.

## About the CKA exam

The CKA is an online, proctored, **performance-based** exam: 15-20 tasks on live clusters in 2 hours, passing score 66%, certification valid for 2 years. The purchase includes one free retake and two killer.sh simulator sessions.

Current curriculum domains and weights:

| Domain | Weight | Topics |
|---|---|---|
| Troubleshooting | 30% | Clusters and nodes, cluster components, resource usage, container output streams, services and networking |
| Cluster Architecture, Installation & Configuration | 25% | RBAC, kubeadm install and lifecycle, HA control plane, Helm and Kustomize, CNI/CSI/CRI, CRDs and operators |
| Services & Networking | 20% | Pod connectivity, NetworkPolicies, Service types and endpoints, Gateway API, Ingress, CoreDNS |
| Workloads & Scheduling | 15% | Deployments and rollouts, ConfigMaps and Secrets, autoscaling, self-healing primitives, admission and scheduling |
| Storage | 10% | StorageClasses and dynamic provisioning, volume types and access modes, PVs and PVCs |

Always confirm the current version, duration, and passing score on the official pages:

- CKA curriculum: https://github.com/cncf/curriculum
- Exam page & candidate handbook: https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/
- Kubestronaut program: https://www.cncf.io/training/kubestronaut/

## Status

**Starting November 2026.** Target exam: January 2027. See [`notes/study-plan.md`](notes/study-plan.md).

## Attribution & license

Notes are built while working through Mumshad Mannambeth's CKA course and expanded with additional depth from the official Kubernetes documentation. They are original write-ups intended for study and sharing; no course material is reproduced. Every diagram is an original PlantUML drawing (source `.puml` committed next to the rendered `.svg`), not a course slide.

This repository is dual-licensed (see [`LICENSE`](LICENSE)):

- **Notes & documentation** (`notes/`, diagrams, this README) — [CC BY 4.0](LICENSE-CC-BY-4.0). Reuse and adapt freely, including commercially, with attribution.
- **Code** (`examples/`, scripts, manifests) — [MIT](LICENSE-MIT).
