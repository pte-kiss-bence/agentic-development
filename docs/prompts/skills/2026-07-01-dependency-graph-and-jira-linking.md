# Dependency-graph skill + Jira dependency-linking upgrade

Generated: 2026-07-01 · Target tool: Claude Code · Category: skills

---

```
You are extending the `pte-openspec-*` planning pipeline in this repository. Two deliverables, plus the schema/convention/example changes that make them reliable. Every design decision below is LOCKED — do not re-open, re-litigate, or "improve" them. Implement exactly as specified.

## Context (carry forward — decisions already made, do not re-decide)

- The pipeline lives in `pte-skills/` (the real files; `.claude/skills/` and `.agents/skills/` are symlinks into it — edit the real files under `pte-skills/`).
- Backlog artifacts: `openspec/backlog/epics/EPIC-<epic-slug>.md` and `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md`.
- Shared rules: `pte-skills/pte-openspec-shared/CONVENTIONS.md`. Glossary: `docs/glossary.md`. Convention: ALL `##` headings are English; body prose is Hungarian.
- Dependencies TODAY are free-text Hungarian prose in the Jira-visible `## Risks and Dependencies` section. That stays as human narrative and is NOT machine-parsed.
- NEW machine-readable source of truth for edges: a dedicated section `## Dependency Edges`, placed INSIDE the `<!-- pipeline-only -->` fence (so it is stripped from the Jira Description, exactly like `## Estimation`).

### The locked edge notation (`## Dependency Edges`)

Each edge is one bullet: an ASCII keyword tag (language-independent so the parser is stable) + a backticked target trace ID. Four tags, edges written from the current card's own perspective:

```
<!-- pipeline-only:start -->
## Dependency Edges
- depends-on: `EPIC-excel-ingest`
- blocks: `STORY-riport-export`
- relates-to: `EPIC-idoszak-lezaras`
- external: Entra ID tenant + szerepkör-hozzárendelés
<!-- pipeline-only:end -->
```

- `depends-on: <TRACE-ID>` — this card waits for the target (target blocks this card).
- `blocks: <TRACE-ID>` — this card blocks the target.
- `relates-to: <TRACE-ID>` — non-blocking coupling.
- `external: <free text>` — an external dependency with no trace ID; graph-only node, never a Jira link.
- BOTH directions (`depends-on` and `blocks`) are allowed. They are inverses; the tooling MUST dedupe an inverse pair into ONE edge (X `depends-on` Y and Y `blocks` X = one edge).
- Tag must start the bullet as `tag: `, followed by a single backticked `TRACE-ID` (except `external:`, which is free text). A dangling target (trace ID not present anywhere in the backlog) MUST produce a warning and be skipped — never crash.

## Deliverable 1 — Author the new skill `pte-openspec-dependency-graph`

Use the `/writing-great-skills` skill to author it (invoke that skill; follow its structure/quality guidance). Skill purpose: read the whole backlog and render a Mermaid dependency flowchart.

Locked behavior:
- Reads every epic + story card, parses ONLY the `## Dependency Edges` sections. Reads no other prose for edges.
- Diagram type: Mermaid `flowchart`. Scope: whole backlog by default; optionally narrow to a single epic via the skill's argument.
- Granularity: two-tier in ONE graph — each epic is a `subgraph` container, its stories are nodes inside it, dependency edges run between story and/or epic nodes. Cross-epic edges are visually highlighted. If a single epic is too large/noisy, fall back to an epic-level overview (epics only, aggregated edges).
- Arrow direction (fixed, include a legend stating it): the arrow means execution order, blocker → blocked. `blocks: Y` on card X draws `X --> Y`. `depends-on: Z` on card X draws `Z --> X`. Every arrow reads "must be done before".
- Edge styling: solid arrow for `blocks`/`depends-on`; dotted for `relates-to`; dashed leaf node for `external:`. Use high-contrast styling and a legend per the `design-doc-mermaid` conventions.
- Cycle detection: the `blocks`/`depends-on` edges MUST form a DAG. Detect cycles, warn prominently, and list the offending cycle. `relates-to` is non-directional and excluded from the cycle check.
- Output artifact: `openspec/backlog/DEPENDENCY_GRAPH.md` — a Markdown file with an embedded ` ```mermaid ` block. Image export to `.svg` is OPTIONAL (only on explicit request), using the `design-doc-mermaid` `scripts/` Python utility.

Role split between the two installed mermaid skills (state this in the skill so it actually uses them):
- `mermaid-diagrams` — the authoritative `flowchart` syntax source (nodes, edges, subgraphs, edge styles).
- `design-doc-mermaid` — its `scripts/` Python utilities (validation + `.mmd`→`.svg` image export), high-contrast styling, and file-organization conventions.

The skill is read-only w.r.t. the cards (it authors no epics/stories). It is optional / on-demand, runnable any time after `pte-openspec-to-stories`, and independent of `pte-openspec-bdd-tests` and `pte-openspec-jira-sync`.

Name the skill `pte-openspec-dependency-graph`. Match the house style of the existing `pte-openspec-*` SKILL.md files (English `##` headings, the same voice, a checklist/steps section, an `EXAMPLE.md` if the siblings have one). All skill BODY prose and any example artifact content stay Hungarian per `CONVENTIONS.md`; the skill's `##` headings and the ASCII edge tags stay English/ASCII.

## Deliverable 2 — Emit the edges from the authoring skills

Update `pte-openspec-to-epics` and `pte-openspec-to-stories` so that, going forward, they author the `## Dependency Edges` section inside the pipeline-only fence on every epic and story card. Derive the edges best-effort from the OpenSpec spec's requirement cross-references and the `## Risks and Dependencies` narrative the skill already writes; a human refines them afterward. The `## Dependency Edges` section is the single source of truth for edges; the `## Risks and Dependencies` prose remains human narrative and is not parsed.

## Deliverable 3 — Record the convention + glossary term

- Add a subsection to `pte-skills/pte-openspec-shared/CONVENTIONS.md` defining the `## Dependency Edges` section, its pipeline-only placement, the four tags, direction-from-the-card's-perspective, inverse-dedup, dangling-target = warn-and-skip, and the ASCII-tag rule.
- Add a glossary term to `docs/glossary.md` for the dependency-edge notation, consistent with the existing trace-ID / pipeline vocabulary entries.

## Deliverable 4 — Upgrade `pte-openspec-jira-sync` to create real Jira issue links

Add a NEW default-on reconcile step (call it step 6c) that runs AFTER all issue keys are assigned (step 5) and alongside the existing key-dependent enrichment (6b). Locked behavior:

- Source: read each card's `## Dependency Edges` section directly (NOT `DEPENDENCY_GRAPH.md`).
- Link-type resolution via `getIssueLinkTypes`, tolerant of localization (a Hungarian project names them e.g. "Blokkolja"/"Kapcsolódik"). Resolve to the canonical English link type; if it cannot be resolved unambiguously, ask with `AskUserQuestion` — mirror the existing issue-type resolution pattern.
- Mapping: `blocks: Y` on X → X **blocks** Y (Jira "Blocks" type, outward). `depends-on: Z` on X → Z blocks X (same "Blocks" type, reversed direction). `relates-to` → "Relates" type. `external:` → skipped (no Jira issue exists).
- Idempotency + inverse-dedup: before creating a link, read the issue's existing `issuelinks` (via `getJiraIssue`) and do not duplicate; X-blocks-Y and Y-is-blocked-by-X are one link. A re-run is a no-op.
- Stale handling: ADDITIVE + drift-report. Never delete a Jira link. If a Jira dependency-link no longer matches any local edge, REPORT it (leave removal to a human) — respect "never clobber the other side's work".
- Field ownership: dependency edges are LOCAL-owned (the pipeline authored them) → pushed Local→Jira. Add a row for them to the field-ownership table. Gate the writes behind the skill's existing write-confirmation step (step 4); do not add a separate opt-in.
- Dangling target (trace ID not in the backlog) → warn and skip, never crash.
- Update the skill's step checklist so 6c appears with a completion criterion, and update the prose to reflect the new step.

## Deliverable 5 — Keep examples truthful + wire the skill in

- Update the `EXAMPLE.md` files of `pte-openspec-to-epics`, `pte-openspec-to-stories`, and `pte-openspec-jira-sync` to show the new `## Dependency Edges` section and the Jira dependency-linking, so no example lies.
- Wire the new skill into the pipeline docs as an OPTIONAL viz step: `pte-skills/README.md`, the `pte-openspec` SKILL.md router, and the CLAUDE.md "offer a right-sized next step" block. Mark it optional; it is not a mandatory chain link.

## Scope locks

- Edit ONLY: the real files under `pte-skills/` (never the `.claude/skills/` or `.agents/skills/` symlinks), `docs/glossary.md`, `CLAUDE.md`, and create `openspec/backlog/DEPENDENCY_GRAPH.md` only if you actually generate a graph as a demonstration (otherwise leave graph generation to the skill at runtime).
- Do NOT touch the devcontainer (`.devcontainer/`), `.claude/settings.json`, `skills-lock.json`, or any script.
- Do NOT add dependencies, change `package.json`, or install anything.

## Forbidden actions

- Do NOT invent new tags, new sections, or a different notation than the one locked above.
- Do NOT parse `## Risks and Dependencies` prose for edges anywhere.
- Do NOT make the Jira link step delete links, and do NOT make it a separate opt-in.
- Do NOT change locked decisions (arrow direction, additive+report, default-on, section name, filename `DEPENDENCY_GRAPH.md`).
- Only make the changes described here. Do not add extra abstractions, files, or refactors beyond what is asked.

## Language

- The skill body prose and ALL product/artifact content (epics, stories, examples, convention text, glossary, DEPENDENCY_GRAPH.md prose, Jira comments) are HUNGARIAN.
- `##` headings and the ASCII edge tags (`depends-on`/`blocks`/`relates-to`/`external`) are English/ASCII.

## Stop conditions & checkpoints

- Before writing any Jira-mutating logic or editing more than one skill, STOP and show a short plan of the exact files you will touch and confirm with me.
- STOP and ask before: deleting any file, editing anything outside the scope-lock list, or adding any dependency.
- After each deliverable, output: ✅ [deliverable] — [files changed]. Then continue to the next.
- Final step: run a self-check — every `##` heading English, edge notation identical across CONVENTIONS/authoring skills/graph skill/jira-sync/examples, no symlink edited, no forbidden change. Report the result.
- Read the sibling files before writing: `pte-skills/pte-openspec-jira-sync/SKILL.md`, `pte-skills/pte-openspec-shared/CONVENTIONS.md`, `pte-skills/pte-openspec-to-epics/SKILL.md` + `EXAMPLE.md`, `pte-skills/pte-openspec-to-stories/SKILL.md` + `EXAMPLE.md`, and both mermaid skills' `SKILL.md`, so your additions match the house style exactly.
```

---

*This prompt is for an agentic tool with real system access. Review the scope locks, forbidden actions, and stop conditions before pasting. Confirm file paths, directories, and permissions match the actual project.*
