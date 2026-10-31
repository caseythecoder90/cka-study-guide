# Project context for Claude

## What this repo is

CKA (Certified Kubernetes Administrator) study notes. Study repository only — no application build. The CKA is one of the five certifications required for CNCF **Kubestronaut** status (KCNA, KCSA, CKA, CKAD, CKS — all active at the same time), which is the end goal behind this repo. Sibling repos: [ckad-exam-prep](https://github.com/caseythecoder90/ckad-exam-prep) (CKAD, done) and [kcna-study-guide](https://github.com/caseythecoder90/kcna-study-guide) (KCNA, done).

The notes follow Mumshad Mannambeth's Udemy course *"Certified Kubernetes Administrator (CKA) with Practice Tests"* (KodeKloud), section by section, and are expanded with the mechanics and context that make the tasks stick.

**The CKA is a hands-on, performance-based exam.** About 2 hours, 15-20 tasks on live clusters in a remote desktop, passing score 66%, open book (kubernetes.io docs only). That puts it closer to the CKAD repo than the KCNA one: speed with `kubectl`, imperative generators, knowing where files live on a node, and a reliable troubleshooting order matter more than precise definitions. **Troubleshooting is the heaviest domain (30%).**

**This repo must be a complete CKA resource on its own.** It is written for anyone preparing for the CKA or working toward Kubestronaut, not only for its author. A reader should never need the sibling repos to pass. Much of the course overlaps the CKAD — those topics still get full chapters here, written from the administrator's view (where the component runs, which file configures it, how it fails), with a closing `## Related notes` section linking the matching CKAD/KCNA chapters for readers collecting the full set. `notes/ckad-crossover.md` maps which topics are review versus new, which sets study pace, not note depth.

- Notes live in `notes/`, grouped into **per-section folders** named `NN-<section-slug>/`, one per course section. Each section folder contains its own numbered chapter notes.
- Each section folder has its own `diagrams/` subfolder alongside the notes. Diagrams live with the section that references them — **not** in a single global folder.
- **Diagrams are original PlantUML drawings, never course screenshots.** Each diagram is a `NN-slug.puml` source rendered to `NN-slug.svg` beside it by `tools/diagrams/render.sh`. Lecture screenshots are reference input only — the instructor's slides are not ours to redistribute.
- `tools/diagrams/` holds the shared PlantUML theme (`cka-theme.puml`) and the render script.
- `notes/study-plan.md` is the **schedule**: week-by-week course plan for November-December 2026, the January 2027 exam window, and the Kubestronaut roadmap. Tick sections off as they are completed.
- `notes/ckad-crossover.md` maps every CKA competency to the related CKAD chapters and flags what is new for a CKAD holder. It is a study-pace guide and the source for each chapter's `## Related notes` links.
- `notes/exam-facts.md` is the **global recall sheet**: paths on a node, ports, default flags, certificate locations, component manifests, version-skew rules. Anything you would otherwise have to hunt for during the exam. Grows with every chapter.
- `notes/commands.md` is the global single-file command reference; `notes/commands/` holds per-topic command files. **`notes/commands/imperative.md`** is the exam-time reference for imperative one-liners and `--dry-run=client -o yaml` generation, as in the CKAD repo.
- `README.md` (repo root) is the public landing page. `examples/` holds runnable manifests and scripts from the labs.

### Section folder ↔ course section mapping

Confirmed against the course outline. **Folder numbers match the course section numbers**, as in the KCNA repo, so there is no `01-` folder.

| Folder | Course section |
|---|---|
| `notes/02-core-concepts/` | 2. Core Concepts |
| `notes/03-scheduling/` | 3. Scheduling |
| `notes/04-logging-and-monitoring/` | 4. Logging & Monitoring |
| `notes/05-application-lifecycle-management/` | 5. Application Lifecycle Management |
| `notes/06-cluster-maintenance/` | 6. Cluster Maintenance |
| `notes/07-security/` | 7. Security |
| `notes/08-storage/` | 8. Storage |
| `notes/09-networking/` | 9. Networking |
| `notes/10-design-and-install-a-cluster/` | 10. Design and Install a Kubernetes Cluster |
| `notes/11-install-with-kubeadm/` | 11. Install "Kubernetes the kubeadm way" |
| `notes/12-helm-basics/` | 12. (2025 Updates) Helm Basics |
| `notes/13-kustomize-basics/` | 13. (2025 Updates) Kustomize Basics |
| `notes/14-troubleshooting/` | 14. Troubleshooting |
| `notes/15-other-topics/` | 15. Other Topics (JSONPath, advanced kubectl) |

Sections 1 (Introduction), 16 (Lightning Labs), 17 (Mock Exams) and the conclusion get no folder: mock-exam and lightning-lab misses feed `exam-facts.md`, `commands/imperative.md`, and the chapter they belong to.

### Notes file layout (the schema)

```
notes/
├── 02-core-concepts/
│   ├── 01-cluster-architecture.md
│   └── diagrams/
│       ├── 01-harbor-analogy.puml   # source
│       ├── 01-harbor-analogy.svg    # rendered, committed
│       └── ...
├── 03-scheduling/
├── ...
├── commands/
│   ├── README.md
│   ├── imperative.md
│   └── <topic>.md
├── commands.md
├── exam-facts.md
├── ckad-crossover.md
├── study-plan.md
└── NOTES-WORKFLOW.md
tools/diagrams/
├── cka-theme.puml         # shared skinparams — every diagram !includes it
└── render.sh              # renders notes/**/diagrams/*.puml → .svg
```

**Rules for new notes (apply this when generating chapters):**
- A new chapter goes in its section folder (`NN-<section>/CC-<chapter-slug>.md`). **Chapter numbers reset at each section** — every section's notes start at `01-...` (or `00-...` for an intro/setup file). Do *not* continue the numbering from the previous section.
- Diagrams referenced by a chapter go in that same section's `diagrams/` subfolder. Diagram numbers run continuously **within a section** and reset per section.
- Image references in markdown use **relative paths from the notes file**: `![Caption](./diagrams/NN-name.svg)`. Never use `../diagrams/...` and never hard-code repo-rooted paths (e.g. `notes/...`) in image links.
- **Diagrams must be embedded in the body to render.** A `companion_diagrams:` list in YAML frontmatter is metadata only and does **not** display the image.
- One topic per file. Don't bundle multiple course lectures into one markdown file unless they are genuinely one topic split across short videos.
- **Self-contained chapters.** Every lecture gets a complete chapter, including topics the CKAD repo already covers. Never replace content with "see the CKAD repo". Re-verify crossover material against current kubernetes.io docs rather than copying the CKAD note, and add the administrator's angle the CKAD note lacked.
- **`## Related notes`** goes after `## References` whenever a sibling repo covers the topic: one bullet per matching chapter, e.g. `- CKAD: [RBAC](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/09-security/06-rbac.md) — the developer's view`. Take the links from `notes/ckad-crossover.md`; verify each target exists. Omit the section when nothing matches. Fill the "CKA chapter" column in `ckad-crossover.md` in the same PR.

### Diagram workflow (PlantUML → SVG)

1. Write `notes/<section>/diagrams/NN-slug.puml`. First line after `@startuml` is always `!include ../../../tools/diagrams/cka-theme.puml` — it pins the Smetana layout engine (no Graphviz needed) and the palette.
2. Render: `tools/diagrams/render.sh notes/<section>/diagrams/NN-slug.puml` (or no args for everything). Commit **both** the `.puml` and the `.svg`.
3. **Verify visually before committing.** Render a PNG to the scratchpad (`java -jar ~/.cache/plantuml/plantuml.jar -tpng -o <scratchpad> file.puml`) and Read it. Check for text overflowing boxes, notes drifting sideways, and unreadable arrow crossings. Re-layout and re-render until clean.
4. Design rules that have worked:
   - Redraw the *concept*, not the slide. Use the course's structure as input; layout, wording, and grouping are ours.
   - `left to right direction` for flows and timelines; default top-to-bottom for hierarchies and side-by-side comparisons.
   - Group with `rectangle "Title" <<group>> { ... }` (`<<groupGood>>` green / `<<groupBad>>` red for contrast). Stack items inside a group with `-[hidden]down->` links; place groups side by side with `-[hidden]right->`.
   - Captions go inside the group as a `label` (hidden-linked below the last item). `note bottom of X` tends to float to the side — avoid it.
   - Emphasis stereotypes: `<<good>>`, `<<bad>>`, `<<accent>>` (filled blue), `<<muted>>`, `<<plain>>`.
   - Sub-text inside a box: `Main line\n<size:11>detail · detail</size>`.
   - Earn the diagram: hierarchies, flows, before/after, comparisons. Tables of values are markdown, not images.

---

## Commit & PR workflow (required for every change)

Every change — even a small notes edit — goes through this flow. Don't commit directly to `main`.

1. **Open a GitHub issue** describing what's being added or changed.
   ```bash
   gh issue create --title "Add notes for <topic>" --body "Covers <what>. Includes diagrams <N>-<M>."
   ```
   Capture the issue number from the output.

2. **Create a feature branch** off `main`:
   ```bash
   git switch -c notes/<chapter-slug>      # for new chapter notes
   git switch -c docs/<short-slug>         # for recall sheets, commands, study plan, or meta docs
   git switch -c fix/<short-slug>          # for corrections
   ```

3. **Commit** on the branch with a clear, conventional message:
   ```bash
   git add <files>
   git commit -m "Add <topic> notes (section NN, chapter CC)"
   ```
   No emojis. No co-author trailer unless explicitly asked. One concise subject line; longer body only if the change needs explanation.

4. **Push the branch** to `origin`:
   ```bash
   git push -u origin HEAD
   ```

5. **Open a PR** with `gh pr create`. The PR body **must** include a GitHub auto-close keyword referencing the issue:
   ```bash
   gh pr create --title "Add <topic> notes" --body "$(cat <<'EOT'
   ## Summary
   - <what changed>

   Closes #<issue-number>
   EOT
   )"
   ```

6. **Merge the PR** (`gh pr merge --squash --delete-branch` is the default). The linked issue closes automatically when the merge reaches `main`.

### Quick sanity check before merging
- Issue linked in PR body via `Closes #N`? ✓
- Branch is feature branch, not `main`? ✓
- Commit message is informative (no "wip", no "update")? ✓
- `exam-facts.md`, `commands.md`, and the matching `commands/<topic>.md` updated for anything new the chapter introduced? ✓
- `study-plan.md` ticked if the chapter finishes a section? ✓
- Chapter readable without the sibling repos; `## Related notes` links verified? ✓

---

## Recall sheet — keep it in sync with every chapter

The CKA rewards knowing *where things are* on a node without searching. **`notes/exam-facts.md`** carries that load and every chapter PR updates it in the same commit:

- File paths: static Pod manifests, PKI directory, kubelet config, kubeconfigs, etcd data dir, CNI config.
- Ports: API server, etcd, kubelet, scheduler, controller manager, NodePort range.
- Flags that matter in a task: `etcdctl` cert flags, `--data-dir`, `kube-apiserver` admission and authorization flags, kubelet `staticPodPath`.
- Rules: version skew, upgrade order, certificate validity, default service CIDRs.

## Commands documentation rules

When a chapter includes `kubectl`/`kubeadm`/`etcdctl`/`crictl`/`systemctl`/shell commands:

- **Always update `notes/commands.md`** (the global reference) under the relevant section.
- **Always update the per-topic file** in `notes/commands/` (create it and add it to `commands/README.md` if none exists).
- **`notes/commands/imperative.md`** is the exam-time reference for imperative one-liners and `--dry-run=client -o yaml` generation — keep it in sync when a new resource is covered.
- Commands are the main event here, as in the CKAD repo. Prefer the fastest correct form, and show the verification command after the change.

---

## Style

- No emojis in notes or commit messages unless the user explicitly asks for them.
- Concise, exam-focused phrasing. No marketing fluff, no padding.
- **Write for a stranger.** Assume the reader has basic Linux and container knowledge but has not read the other repos or taken the course. Define a term the first time a section uses it; link back to the earlier chapter in this repo rather than to a sibling repo.
- **Lean notes — no scaffolding.** Do NOT add "Why this is in CKA" preambles, "Key takeaways"/"TL;DR" summaries, "Open threads" checklists, instructor narration, or personal anecdotes. Keep commands, YAML, diagrams, decision tables, file paths, and the exam-relevant "why".
- **Prefer discovery over memorization.** For fields with no imperative generator, show how to recall them with `kubectl explain <path>`, `--help`, or which kubernetes.io page to open in the exam browser.
- **Troubleshooting order is load-bearing.** For any component, write down the order to check it: status → events/logs → config file → the process on the node (`systemctl`, `journalctl`, `crictl`).
- **`## Exam angle` closes every chapter** (before References): 3-6 bullets on what a task on this topic looks like, the fastest path to solve it, how to verify, and the trap that costs points (wrong context, wrong namespace, editing a live static Pod instead of its manifest, ...).
- **References section.** Each chapter ends with a `## References` section of 2-4 canonical, verified `kubernetes.io` links (or the official docs of the tool discussed: etcd.io, helm.sh, kubectl.docs.kubernetes.io, gateway-api.sigs.k8s.io). These are the pages you can open during the exam — say which section of the page holds the YAML you need. Verify URLs before adding them — never guess a doc URL.
- Prefer editing existing files over creating new ones; keep new files focused (one topic per file).
