---
name: pte-openspec-to-epics
description: Generate or update Agile Epics (Atlassian/SAFe-style) from OpenSpec specs. Use when the user wants epics or a strategic backlog decomposed from OpenSpec requirements or a change's delta spec (mint mode), or wants a change request folded into the epic it already affects (reconcile mode).
---

An OpenSpec spec is detailed behaviour: each `### Requirement` holds `#### Scenario` blocks of `WHEN`/`THEN` bullets. This skill rolls that detail up into **Agile Epics** — one markdown file per epic, each a value-oriented strategic container that breaks down into user stories.

`lean` is the governing word: an epic is a guide to *value*, not a task bucket. It states a hypothesis, names its scope and non-goals, and splits into **vertical-slice** stories cut by user value or journey — never by technical layer. Note: "Epic" is not a Scrum Guide artifact (the Scrum Guide defines Product Backlog Items); the format below follows Atlassian / SAFe best practice. A complete worked transformation — spec in, epic out, with IDs and the traceability map — is in [`EXAMPLE.md`](EXAMPLE.md).

Shared pipeline conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, INVEST, and diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md) (terms in [`docs/glossary.md`](../../docs/glossary.md)); read it before writing.

## Modes — mint vs reconcile

This skill runs in one of two modes; pick by whether the change's capability already has an epic on disk. A **change request modifies existing behaviour**, so reconcile is the common case for changes; mint is for genuinely new capabilities.

- **Mint (greenfield)** — the spec/delta describes a capability with **no existing epic**. Mint new epic(s) from scratch, per the anatomy and steps below. This is the original behaviour.
- **Reconcile (update)** — a **change's delta spec** whose capability **already has an epic**. Do **not** mint a new epic: resolve the existing epic, reuse its `EPIC-…` verbatim, and fold the delta into it — mint new `STORY-…` rows for the delta's *added* requirements/scenarios, **extend an existing story's** *Forrás-spec hivatkozás* coverage when a *modified* scenario already belongs to one (never mint a duplicate for it), and touch *MVP and Out of Scope* only if the change moves the boundary. Everything the delta does not touch stays byte-for-byte (diff-don't-clobber).

Resolve the mode in step 1: search the epic dir for an epic whose *Forrás-spec hivatkozás* rows trace to the delta's capability. One hit → reconcile it. Several plausible → **AskUserQuestion**. None → mint; but if the change is a single small item not worth an epic, hand off to `pte-openspec-to-stories`'s **epic-less mode** instead of minting a one-story epic.

## Source mapping

| OpenSpec | Agile Epic |
|----------|------------|
| `openspec/specs/<capability>/spec.md` (or a change delta spec) | one or more Epic `.md` files (split when >~10 stories) |
| capability theme / a coherent group of Requirements | one Epic + its `EPIC-<epic-slug>` |
| `### Requirement: <name>` (its SHALL text) | one or more user stories, split by value |
| `#### Scenario: <name>` (`WHEN`/`THEN`) | a story's high-level acceptance criteria |
| the spec's purpose / overview | the epic's *Description* + *Problem / Solution* |

## Trace ID convention (this skill OWNS it)

This skill **MINTS** the pipeline's trace IDs; `pte-openspec-to-stories` and `pte-openspec-bdd-tests` only consume them (except `pte-openspec-to-stories`'s epic-less mode, which mints its own capability-namespaced `STORY-…`). In **reconcile** mode this skill reuses the existing epic's `EPIC-…` **verbatim** and mints only the new `STORY-…` the delta adds — it never re-mints an epic ID. The full ID grammar, ownership, and the *Forrás-spec hivatkozás* map contract live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md). This skill's specific job:

- Assign one `EPIC-<epic-slug>` per epic and one `STORY-<epic-slug>-<requirement-slug>` per story; add a stable `-<aspect-slug>` when several stories split one requirement.
- The requirement→story split is a judgement call, NOT derivable from the spec alone — that is why this skill emits the traceability map (below) and why downstream skills read it rather than re-derive. The ID grammar is stable; the mapping is what they need.

## Epic file anatomy

Section **headings are English** (the new schema — see the golden sample in [`EXAMPLE.md`](EXAMPLE.md)); the **content stays Hungarian** (per `CONVENTIONS.md`). Each epic file carries these sections, in order:

- **Cím** — the H1: `# <epic name> · EPIC-<epic-slug>`.
- **`## Description`** — narrative from the spec's purpose/overview: what the capability is and who it serves.
- **`## Persona`** — a bulleted list of the affected roles/stakeholders, each with a one-line note on what they do in this capability.
- **`## E2E Scenario`** — the epic's **measurable hypothesis** in prose: `Ha [X], akkor [Y] a [Z] csoportnak, mérve [W]-vel.` This section absorbs the old *Hipotézis* — keep the `mérve …` measurability.
- **`## Problem / Solution`** — the problem the spec solves and the solution shape, as **two stacked paragraphs**: a `P: …` paragraph and a **separate** `S: …` paragraph with a **blank line between them**, so they render one under the other and never merge into a single `P: … S: …` paragraph.
- **`## Cross-cutting Concerns`** — the concerns that cut **across** the standard, documented behaviour and must not be overlooked: special environment, security, performance, monitoring, logging, auditability, accessibility, localization, data-handling specifics. **Fill it with the real, spec-derived concerns** for this epic (Hungarian) — do **not** emit the rubric text itself. Include a concern only when the spec actually raises it; never pad.
- **`## MVP and Out of Scope`** — the in-scope MVP bullets and the explicit **Out of Scope** bullets (replaces the old *Hatókör és nem-célok*).
- **`## Success metrics`** — measurable success metrics.
- **`## Risks and Dependencies`** — dependencies + risks (replaces *Függőségek és kockázatok*).
- **`## High Level Acceptance Criteria`** — derived from the Requirements' `#### Scenario` blocks.
- **`## Estimation`** — a **rollup** of the epic's child-story estimates: the Σ of their `Eβ` ideal engineer-hours and Σ Story Points, per the epic-rollup rule in [`../pte-openspec-shared/ESTIMATION.md`](../pte-openspec-shared/ESTIMATION.md). It is rendered in the **same two-line shape as a story card** — `- **Becsült munkaóra:** <Σ Eβ> ó (≈ <ideal days> ideális nap)` and `- **Story Point:** <Σ SP>` (bare integer, no `Σ` prefix, no story-count suffix; the raw `O`/`M`/`P`/`σ` are never printed). Because the estimate lives on the stories (authored by `pte-openspec-to-stories`, which runs *after* this skill), on first mint this section is a **placeholder** — `_(rollup: minden story esztimálása után)_` — that the **estimation-rollup pass** (below) fills once the child cards exist and are estimated. If any child story lacks an estimate, mark the rollup incomplete (`⚠ nem minden story esztimált`) rather than guessing a total.

Then a **pipeline-only block** — the pipeline's internal contract, **hidden from Jira** (`pte-openspec-jira-sync` strips the whole fenced region before push; see `CONVENTIONS.md`). Wrap it in the fence and keep its Hungarian headings:

````markdown
<!-- pipeline-only:start -->
## User story-k
…
## Forrás-spec hivatkozás
…
<!-- pipeline-only:end -->
````

- **`## User story-k`** (pipeline-only) — typically ~3–10 stories. Each starts with its `STORY-…` ID, then `Mint [szerep], szeretnék [cél], hogy [érték]`, with acceptance criteria mapped from the matching `#### Scenario` `WHEN`/`THEN`. Fewer than 3 is fine when the capability is genuinely small — never pad with invented stories.
- **`## Forrás-spec hivatkozás`** (pipeline-only) — the traceability table, the contract the downstream skills consume: `pte-openspec-to-stories` and `pte-openspec-bdd-tests` read **this** table (the story cards no longer carry their own trace row) to learn the requirement→story split and each story's `#### Scenario` coverage. One row per story, titles **verbatim**:

  | Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
  |----------|---------|--------------------------|----------------------------|

## Steps

Copy this checklist and tick each item — the verify step is exhaustive, not a glance:

```
- [ ] 1. Source spec file set named
- [ ] 2. Every Requirement and Scenario enumerated
- [ ] 3. Requirements grouped into lean Epics; EPIC IDs assigned
- [ ] 4. Each Epic split into vertical-slice stories (each INVEST-conform); STORY IDs + Scenario coverage assigned
- [ ] 5. Epic files authored (full Hungarian anatomy + hypothesis + traceability map + ## Estimation rollup placeholder)
- [ ] 6. Files written to the output dir (diffed, not clobbered)
- [ ] 7. Every Requirement and Scenario verified accounted for; gaps reported
- [ ] 8. (rollup pass, when child stories are estimated) ## Estimation filled from Σ child Eβ + Σ SP, or marked incomplete if any child is unestimated
```

1. **Resolve the source specs and pick the mode.** A change's delta (`openspec list --json` → pick → `openspec/changes/<id>/specs/**/spec.md`) or a main spec (`openspec/specs/<capability>/spec.md`). If the input is vague, list the options with **AskUserQuestion**. If the user names a store, pass `--store <id>` on `openspec` commands, as the other `openspec-*` skills do. Then resolve **mint vs reconcile** (see *Modes*): scan the epic dir for an epic whose *Forrás-spec hivatkozás* rows trace to this delta's capability — one hit → reconcile; several → **AskUserQuestion**; none → mint. Completion: the exact file set is named and the mode is fixed.

2. **Enumerate every Requirement and Scenario** in those files — name, SHALL text, and each `WHEN`/`THEN` bullet. Completion (exhaustive): every `### Requirement` and every `#### Scenario` in the source is on the list; none invented, none dropped.

3. **Group into lean Epics.** Cluster Requirements into epics by user value (a coherent capability or journey), not by technical phase. Assign each an `EPIC-<epic-slug>`. Apply the size rule: an epic typically holds ~3–10 stories; if a grouping would exceed ~10, treat it as a signal to split into two or more epics. Fewer than 3 is a soft signal only — acceptable for a small capability, never a reason to invent stories. Completion (exhaustive): every Requirement belongs to exactly one Epic, or is explicitly listed out of scope.

4. **Decompose each Epic into vertical-slice stories — INVEST-conform.** Split by user value/journey; assign each story a requirement-anchored `STORY-…` ID and record which `#### Scenario`s it covers. Cut the split so **every story satisfies INVEST** (Independent, Negotiable, Valuable, Estimable, Small, Testable) — the epic owns the split, and `pte-openspec-to-stories` enforces INVEST as a hard gate downstream, so a slice that cannot be made INVEST-conform must be re-cut here, not passed on. Completion (exhaustive): every Scenario is covered by exactly one story (or explicitly out of scope), and every story is INVEST-conform.

5. **Author each Epic** with the full anatomy above: English headings, Hungarian content, the `Ha … akkor … mérve …` hypothesis in *E2E Scenario*, filled *Cross-cutting Concerns*, acceptance criteria derived from the Scenarios, the `## Estimation` **rollup placeholder** (`_(rollup: minden story esztimálása után)_` — filled later by the rollup pass, since the per-story estimates don't exist yet), and — inside the `<!-- pipeline-only -->` fence — the *User story-k* list and the *Forrás-spec hivatkozás* map table with **verbatim** Requirement and Scenario titles. Completion: every anatomy section is present (Estimation as a placeholder) and the map table covers every story.

6. **Write the files** to the output directory (default `openspec/backlog/epics/`, configurable — see *Output location* in `CONVENTIONS.md`), one file per epic, named `EPIC-<epic-slug>.md` (the `EPIC-` prefix gives searchability parity with the `STORY-…` cards; the bare `<epic-slug>` still names the story directory). Diff before overwriting — never clobber hand-edited content. Completion: each epic exists as its own file under the output dir.

7. **Verify exhaustively.** Every source Requirement and Scenario is accounted for (covered by a story or explicitly out of scope); no Epic exceeds ~10 stories (split if it does); every story has a unique `STORY-…` ID and acceptance criteria; every map-table title matches the spec verbatim; the *User story-k* list and *Forrás-spec hivatkozás* map sit inside the `<!-- pipeline-only:start -->`/`<!-- pipeline-only:end -->` fence, and every non-pipeline section uses its English heading. Report any Requirement or Scenario you could not place cleanly rather than guessing.

### Reconcile mode — how the steps deviate

When step 1 fixed **reconcile**, steps 3–7 fold the delta into the resolved epic instead of minting a new one:

- **Step 3–4 (grouping/decomposition):** the epic already exists, so don't re-group. Scope the work to the delta only — for each **added** Requirement/Scenario, mint a new `STORY-…` under the existing `EPIC-…` (verbatim); for each **modified** Scenario, find the story that already covers it and extend that row's coverage rather than minting a duplicate; for a **removed** Scenario, drop it from its story's map row (and the story itself if it empties). If the delta pushes the epic past ~10 stories, flag a split rather than silently overflowing.
- **Step 5 (author):** edit the existing epic file — extend *User story-k* and the *Forrás-spec hivatkozás* map (inside the pipeline-only fence) with the new/changed rows, and adjust *MVP and Out of Scope* / *Success metrics* only where the change actually moves them. Leave every untouched section byte-for-byte.
- **Step 6 (write):** diff-don't-clobber is load-bearing here — you are editing a hand-groomed epic in place, not rewriting it.
- **Step 7 (verify):** every delta Requirement/Scenario is covered by a story row (new or extended); the `EPIC-…` is unchanged; no `STORY-…` was duplicated for a modified scenario; sections the delta did not touch are unchanged.

### Estimation-rollup pass — filling the epic total once the stories are estimated

The epic's `## Estimation` is a **rollup**, and the per-story estimates it sums are minted *downstream* by `pte-openspec-to-stories` — so it cannot be totalled at mint time. Run this pass on the epic **after** its story cards are authored and estimated (a targeted re-run of this skill, diff-don't-clobber — it touches only the `## Estimation` section):

- **Read every child card.** For the epic, enumerate its `openspec/backlog/stories/<epic-slug>/STORY-…md` cards and read each one's `## Estimation` (`Eβ` hours + SP).
- **Gate on completeness.** Only total when **every** child story carries an estimate. If any child is missing one, write `⚠ nem minden story esztimált` in the epic's `## Estimation` (naming which stories are unestimated) instead of a total — never guess a number for a missing child.
- **Sum and write.** When complete, fill `## Estimation` with Σ `Eβ` ideal engineer-hours and Σ SP across the children (per the epic-rollup rule in `ESTIMATION.md`), replacing the placeholder. Leave every other epic section byte-for-byte. Completion: the epic total equals the sum of its children, or is explicitly marked incomplete.
