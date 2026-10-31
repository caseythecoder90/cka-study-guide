# Study plan

> The schedule for this repo: the course in November and December 2026, the exam in January 2027, then the rest of the Kubestronaut set by April 2027. Tick a section when its notes are merged, not when the videos are watched.

## Kubestronaut roadmap

Kubestronaut needs all five certifications **active at the same time**. Each is valid for 2 years from the pass date (check the date on each certificate), so the earliest expiry sets the deadline for the last exam.

| Cert | Format | Status | Target |
|---|---|---|---|
| CKAD | Hands-on | Done — [ckad-exam-prep](https://github.com/caseythecoder90/ckad-exam-prep) | — |
| KCNA | Multiple choice | Done — [kcna-study-guide](https://github.com/caseythecoder90/kcna-study-guide) | — |
| **CKA** | Hands-on | **In progress — this repo** | **January 2027** |
| CKS | Hands-on | Not started. **Requires a passed CKA** before you can sit it | March 2027 |
| KCSA | Multiple choice | Not started. Largely a theory subset of CKS — take it right after | April 2027 |

Doing CKS before KCSA is deliberate: KCSA's 4C's, threat modelling, Pod Security Standards and supply-chain topics are all covered in more depth by CKS, so KCSA becomes a two-week review instead of a new course.

Fill in the actual pass dates here so the expiry window is visible:

| Cert | Passed | Expires |
|---|---|---|
| CKAD | | |
| KCNA | | |
| CKA | | |
| CKS | | |
| KCSA | | |

## Budget

- Course: about 25 h 45 min of video across sections 1-17 (lengths below). With the KodeKloud labs and complete notes for every section, plan on about **3-3.5x** that: roughly **80-90 hours** over 9 weeks, so **9-10 hours a week**.
- Every section gets complete notes, including the ones that overlap the CKAD, so the repo works for readers who never took the CKAD. Crossover sections (Core Concepts, Application Lifecycle Management, Storage, Helm, Kustomize) still go faster: the material is familiar, the labs are quick, and the CKAD notes are a starting outline to re-verify and expand. See [`ckad-crossover.md`](ckad-crossover.md).
- New-to-you sections (Cluster Maintenance, Security, Networking, kubeadm install, Troubleshooting) get the extra time.
- Writing full notes for crossover sections adds hours. If a week runs over, keep the labs on schedule and let the notes trail by a few days. The exam date depends on the labs.
- Linux Foundation exam vouchers are usually discounted around Black Friday / Cyber Monday. A CKA + CKS bundle bought then covers the next two exams; a voucher gives 12 months to schedule.

### Course sections

| # | Section | Lectures | Video |
|---|---|---|---|
| 1 | Introduction | 5 | 12 min |
| 2 | Core Concepts | 44 | 3 h 39 min |
| 3 | Scheduling | 38 | 3 h 02 min |
| 4 | Logging & Monitoring | 7 | 14 min |
| 5 | Application Lifecycle Management | 34 | 2 h 22 min |
| 6 | Cluster Maintenance | 13 | 46 min |
| 7 | Security | 45 | 4 h 04 min |
| 8 | Storage | 17 | 1 h 05 min |
| 9 | Networking | 35 | 3 h 15 min |
| 10 | Design and Install a Kubernetes Cluster | 5 | 32 min |
| 11 | Install "Kubernetes the kubeadm way" | 6 | 44 min |
| 12 | (2025 Updates) Helm Basics | 11 | 50 min |
| 13 | (2025 Updates) Kustomize Basics | 22 | 1 h 30 min |
| 14 | Troubleshooting | 12 | 1 h 13 min |
| 15 | Other Topics | 4 | 12 min |
| 16 | Lightning Labs | 2 | 1 min (the labs themselves take far longer) |
| 17 | Mock Exams | 6 | 2 h 05 min (plus about 2 h per mock exam attempt) |

Video length understates the hands-on sections: Cluster Maintenance and Troubleshooting are short videos with long labs, so the schedule gives them full weeks.

## Course schedule — November and December 2026

Weeks run Monday to Sunday. Section numbers match the course and the note folders (`notes/02-core-concepts/`, ...). Video per week is in brackets; budget about 3x that for labs and notes.

| Week | Dates | Sections | Crossover | Done |
|---|---|---|---|---|
| 1 | Nov 2 - Nov 8 | 1 Introduction, **2 Core Concepts** (3 h 51 min) | Mostly review, but written from the administrator's view: what runs where, kube-proxy, kubelet | [ ] |
| 2 | Nov 9 - Nov 15 | **3 Scheduling**, **4 Logging & Monitoring** (3 h 16 min) | Mostly review. New: static Pods, multiple schedulers, scheduler profiles, PriorityClasses | [ ] |
| 3 | Nov 16 - Nov 22 | **5 Application Lifecycle Management**, **6 Cluster Maintenance** (3 h 08 min) | ALM is review (new: autoscaling as admin). Cluster Maintenance is **new**: drain/cordon, kubeadm upgrade, etcd backup and restore — short videos, long labs | [ ] |
| 4 | Nov 23 - Nov 29 | **7 Security** part 1 — TLS, certificates, Certificates API, kubeconfig (about 2 h) | **New.** Lighter week (Thanksgiving Nov 26) | [ ] |
| 5 | Nov 30 - Dec 6 | **7 Security** part 2 — API groups, RBAC, ServiceAccounts, image security, security contexts, NetworkPolicies, CRDs; **8 Storage** (about 3 h) | RBAC, NetworkPolicies and Storage are review | [ ] |
| 6 | Dec 7 - Dec 13 | **9 Networking** (3 h 15 min) | **New.** Linux networking, CNI, Pod/Service networking, CoreDNS, Ingress, Gateway API | [ ] |
| 7 | Dec 14 - Dec 20 | **10 Design and Install**, **11 kubeadm**, **12 Helm**, **13 Kustomize** (3 h 36 min) | kubeadm is **new** — build a real cluster (the [k8s-cluster-hetzner](https://github.com/caseythecoder90/k8s-cluster-hetzner) repo is a ready place to practice). Helm and Kustomize are review | [ ] |
| 8 | Dec 21 - Dec 27 | **14 Troubleshooting** (1 h 13 min) | **New and 30% of the exam.** Application, control plane, worker node, networking failures. Short video, so the holiday week still fits — do the labs twice | [ ] |
| 9 | Dec 28 - Jan 3 | **15 Other Topics**, **16 Lightning Labs** | JSONPath partly review. Lightning labs are timed: treat them as a first exam rehearsal | [ ] |

## Exam prep — January 2027

| Week | Dates | Plan | Done |
|---|---|---|---|
| 10 | Jan 4 - Jan 10 | Course **mock exams** 1-3, timed. Every miss goes into the matching chapter, `exam-facts.md`, or `commands/imperative.md`. Re-run the Troubleshooting and Cluster Maintenance labs | [ ] |
| 11 | Jan 11 - Jan 17 | **killer.sh session 1** (two sessions come with the exam purchase; each stays open 36 hours). Fix weak spots. Drill: etcd backup/restore, kubeadm upgrade, broken kubelet, broken control-plane static Pod — until each is under 8 minutes | [ ] |
| 12 | Jan 18 - Jan 24 | **killer.sh session 2**, then **sit the exam** at the end of this week | [ ] |
| — | Jan 25 - Jan 31 | Buffer. The exam purchase includes one free retake | [ ] |

## After the CKA

| Window | Plan |
|---|---|
| Feb 1 - Mar 31, 2027 | CKS course and exam. New repo following this pattern (`cks-study-guide`) |
| April 2027 | KCSA review and exam. Kubestronaut application |

## Daily habit during the course

- Watch a lecture, then do its lab **before** writing the note. The note records what the lab taught, not what the slide said.
- Every new command goes into `commands.md` the same day.
- Once a week, one timed task from a previous section without notes — spaced recall is what the exam measures.
