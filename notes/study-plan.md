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

- Course: about 26 hours of video. With the KodeKloud labs and complete notes for every section, plan on about **3-3.5x** that: roughly **80-90 hours** over 9 weeks, so **9-10 hours a week**.
- Every section gets complete notes, including the ones that overlap the CKAD, so the repo works for readers who never took the CKAD. Crossover sections (Core Concepts, Application Lifecycle Management, Storage, Helm, Kustomize) still go faster: the material is familiar, the labs are quick, and the CKAD notes are a starting outline to re-verify and expand. See [`ckad-crossover.md`](ckad-crossover.md).
- New-to-you sections (Cluster Maintenance, Security, Networking, kubeadm install, Troubleshooting) get the extra time.
- Writing full notes for crossover sections adds hours. If a week runs over, keep the labs on schedule and let the notes trail by a few days. The exam date depends on the labs.
- Linux Foundation exam vouchers are usually discounted around Black Friday / Cyber Monday. A CKA + CKS bundle bought then covers the next two exams; a voucher gives 12 months to schedule.

## Course schedule — November and December 2026

Weeks run Monday to Sunday. Section names follow the course outline and the provisional folder mapping in [`.claude/CLAUDE.md`](../.claude/CLAUDE.md).

| Week | Dates | Sections | Crossover | Done |
|---|---|---|---|---|
| 1 | Nov 2 - Nov 8 | Introduction, **Core Concepts**, **Scheduling** | Mostly review. New: static Pods, multiple schedulers, scheduler profiles, PriorityClasses | [ ] |
| 2 | Nov 9 - Nov 15 | **Logging & Monitoring**, **Application Lifecycle Management** | Mostly review. New: autoscaling as admin (metrics-server, HPA/VPA) | [ ] |
| 3 | Nov 16 - Nov 22 | **Cluster Maintenance** | **New.** OS upgrades (drain/cordon), kubeadm cluster upgrade, etcd backup and restore | [ ] |
| 4 | Nov 23 - Nov 29 | **Security** part 1 — TLS, certificates, Certificates API, kubeconfig | **New.** Lighter week (Thanksgiving Nov 26) | [ ] |
| 5 | Nov 30 - Dec 6 | **Security** part 2 — RBAC, ServiceAccounts, image security, security contexts, NetworkPolicies; **Storage** | RBAC and NetworkPolicies are review; Storage is review | [ ] |
| 6 | Dec 7 - Dec 13 | **Networking** | **New.** Linux networking, CNI, Pod/Service networking, CoreDNS, Ingress, Gateway API | [ ] |
| 7 | Dec 14 - Dec 20 | **Design and Install**, **Install with kubeadm**, Helm and Kustomize basics | kubeadm is **new** — build a real cluster (the [k8s-cluster-hetzner](https://github.com/caseythecoder90/k8s-cluster-hetzner) repo is a ready place to practice). Helm/Kustomize are review | [ ] |
| 8 | Dec 21 - Dec 27 | **Troubleshooting** | **New and 30% of the exam.** Application, control plane, worker node, networking failures. Lighter week (holidays) | [ ] |
| 9 | Dec 28 - Jan 3 | **Other Topics** (JSONPath, advanced kubectl), **Lightning Labs** | JSONPath partly review | [ ] |

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
