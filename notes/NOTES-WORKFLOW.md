# Notes workflow — turning lectures into chapters

How chapters in this repo get written. Same mechanics as the CKAD and KCNA repos, with the CKA-specific additions: self-contained chapters with `## Related notes`, the node-paths recall sheet, and `## Exam angle` written for hands-on tasks.

## Lab first, note second

Watch the lecture, do the KodeKloud lab, then write. The chapter records what the lab taught — the command that worked, the file that had to change, the error that showed what was wrong — not what the slide said.

## Every chapter stands alone

This repo is meant to be a complete CKA resource for anyone, so a reader should never need the CKAD or KCNA repos to pass. Topics the CKAD already covered still get a full chapter here:

- Write the complete topic, re-verified against current kubernetes.io docs — do not copy the CKAD note.
- Add the administrator's view the CKAD note lacked: where the component runs, which file configures it, how it fails, how to fix it on a node.
- Close with `## Related notes` (after `## References`) linking the matching sibling-repo chapters, taken from [`ckad-crossover.md`](ckad-crossover.md). Those links are for Kubestronaut readers who want the developer's or the conceptual angle, never a substitute for content.

## Diagrams are ours, not the course's

Lecture screenshots are **input**: they tell us which concepts the lecture emphasized and how it structured them. They are never committed. Every diagram in `diagrams/` is an original PlantUML drawing (`NN-slug.puml`) rendered to SVG (`NN-slug.svg`) with the shared theme in `tools/diagrams/`. Redraw the concept, not the slide.

```bash
tools/diagrams/render.sh                                   # render everything
tools/diagrams/render.sh notes/<section>/diagrams/NN-x.puml # render one
```

Commit both the `.puml` and the `.svg`. Preview by rendering a PNG into the scratchpad and looking at it before committing.

## The one rule that makes images actually show up

Listing a diagram in YAML frontmatter (`companion_diagrams:`) is **metadata only — it never renders**. An image appears only if it is embedded in the body with a relative path:

```markdown
![Short caption](./diagrams/03-etcd-backup-restore.svg)
```

Always `./diagrams/...` — never `../diagrams/...`, never repo-rooted paths like `notes/...`.

## Numbering rules

- Chapter files: `<CC>-<chapter-slug>.md`, one topic per file. Chapter numbers reset to `01` (or `00` for intro/setup) at the start of each section folder.
- Diagrams: `<NN>-<diagram-slug>.puml` + `.svg` in the section's own `diagrams/` subfolder. Diagram numbers run continuously **within a section** and reset per section.
- To resume: look in the section folder for the highest existing `CC-` file and the highest `NN-` in `diagrams/`, then add 1 to each.

## Per-chapter checklist

1. Chapter file in the right section folder with the next `CC-` number. Complete on its own — no "see the CKAD repo" in place of content.
2. Diagrams written as `.puml`, rendered, previewed, and embedded in the body.
3. Commands with the verification step after each change; file paths written out in full.
4. `## Exam angle` — 3-6 bullets: what the task looks like, the fastest path, how to verify, the trap.
5. `## References` — 2-4 verified kubernetes.io links, saying where on the page the YAML lives. Then `## Related notes` with verified sibling-repo links, if any match.
6. New paths, ports, flags, defaults added to `exam-facts.md`.
7. Commands added to `commands.md`, the per-topic file in `commands/`, and `commands/imperative.md` if there is a generator.
8. If the section is finished: tick it in `study-plan.md`, fill its "CKA chapter" column in `ckad-crossover.md`, and update the README contents table.

---

## Section-start prompt

> You are helping me build CKA study notes from lecture screenshots and lab output I upload. Read `.claude/CLAUDE.md` in this repo and follow its rules exactly — in particular, write every chapter as a complete standalone resource (even where the CKAD repo covers the topic), end it with `## Related notes` links from `notes/ckad-crossover.md`, and draw original PlantUML diagrams (rendered to SVG via `tools/diagrams/render.sh`, previewed before committing) instead of reusing my screenshots.
>
> For this section: section folder = `<fill in>`, start chapter numbering at `<CC>`, start diagram numbering at `<NN>`. First lecture: `<topic>`.
>
> Confirm the folder, the course section name (and correct the provisional mapping in `.claude/CLAUDE.md` if it differs), and the starting numbers, then wait for my first lecture.

## Mid-section continuation prompt

> Continuing CKA notes for section `<SECTION-FOLDER>`. Same rules as `.claude/CLAUDE.md`: complete standalone chapters with `## Related notes`; one topic per file, `<CC>-<slug>.md`; original PlantUML diagrams in `./diagrams/<NN>-<slug>.puml` rendered to `.svg`, previewed, and embedded in the body; commands with verification; `## Exam angle` then `## References`; update `exam-facts.md`, `commands.md` and `commands/`. Resume at chapter `<CC>` and diagram `<NN>`.

---

## Diagram conventions

- **Tool:** PlantUML (`~/.cache/plantuml/plantuml.jar`, Java) with the built-in Smetana layout engine — no Graphviz install needed. Output SVG.
- **Theme:** every `.puml` starts with `!include ../../../tools/diagrams/cka-theme.puml`. Light background, Arial, Kubernetes blue `#326CE5` primary, red `#DC2626` for failures, green `#16A34A` for the fixed state, amber notes.
- **Stereotypes available:** `<<group>>` / `<<groupGood>>` / `<<groupBad>>` for containers; `<<good>>` / `<<bad>>` / `<<accent>>` / `<<muted>>` / `<<plain>>` for boxes.
- **Layout tricks:** `-[hidden]down->` to stack items inside a group, `-[hidden]right->` to put groups side by side, `label` for captions inside a group, `left to right direction` for flows, `\n<size:11>...</size>` for sub-text in a box.
- **Earn the diagram:** control-plane layouts, request paths, upgrade sequences, packet flows, troubleshooting decision trees. Tables of values are markdown, not images.
