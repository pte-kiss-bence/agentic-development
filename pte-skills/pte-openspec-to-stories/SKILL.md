---
name: pte-openspec-to-stories
description: Expand an OpenSpec epic's stories into detailed, refinement-ready Agile story cards — or author standalone cards straight from a small change's delta spec (epic-less mode). Use when the user wants refinement-ready story cards from an epic produced by pte-openspec-to-epics, or wants a small standalone change turned into one or a few cards without creating an epic (epic-less mode).
---

An Agile epic from `pte-openspec-to-epics` already lists its stories — each a `STORY-…` ID, a `Mint … szeretnék … hogy …` line, and high-level acceptance criteria in its *User Stories* section. This skill expands **each** of those stories into a full, **refinement-ready** Agile story card — one markdown file per story.

`refinement-ready` is the governing word: a card is what a team can pull into planning — a clear user-value statement (`## User Story`), the value slice (`## Context`), testable acceptance criteria as declarative Gherkin (`## BDD Test`, filled by `pte-openspec-bdd-tests`), and its risks/dependencies. The schema is lean by design — it never invents behaviour: the acceptance criteria are the `#### Scenario`s the epic's map assigns to the story, traced back to the spec and expressed as the story's BDD test.

The story card is the pipeline's **smallest unit and the home of its own BDD test(s)**: the card carries a `BDD Test` section that `pte-openspec-bdd-tests` fills in place with declarative Gherkin. This skill authors that section as an explicit placeholder — it does **not** write Gherkin itself.

Shared pipeline conventions — output language, trace-ID grammar, the *Source Spec Reference* map contract, source-spec resolution, and diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md); read it before writing. A complete worked transformation is in [`EXAMPLE.md`](EXAMPLE.md).

## Role in the pipeline

The **primary input is the epic file(s)** produced by `pte-openspec-to-epics`, not the raw spec. The epic owns the story set and the IDs; this skill only expands each story. The spec is the **traced-through source** for the acceptance-criteria detail: for each story, follow the epic's *Source Spec Reference* map to the exact `### Requirement` / `#### Scenario` blocks and expand their `WHEN`/`THEN`.

A card embodies the **3 C's**: it is the *Card*, it stands in for a *Conversation*, and its *BDD Test* (the Gherkin scenarios `pte-openspec-bdd-tests` fills) is the *Confirmation*.

## Source mapping

| Agile Epic (`pte-openspec-to-epics`) | Story card |
|--------------------------------------|-----------|
| one *User Stories* entry (`STORY-…` + `Mint …` line) | one story card file |
| the story's `STORY-…` ID + parent `EPIC-…` | **Cím** trace header (verbatim) |
| the epic's `Mint … szeretnék … hogy …` line | first line of **`## User Story`** (verbatim) |
| the `#### Scenario`s the *Source Spec Reference* row assigns | the **`## BDD Test`** placeholder (Gherkin filled later by `pte-openspec-bdd-tests` — that Gherkin **is** the acceptance criteria) |
| the epic's *Source Spec Reference* row | **stays in the epic** — the card carries no trace row (epic-less: self-carried, pipeline-only) |

## Story card anatomy

Section **headings are English** (the new schema — see the golden sample in [`EXAMPLE.md`](EXAMPLE.md)); the **content stays Hungarian** (per `CONVENTIONS.md`). Each card carries these sections, in order — the schema is deliberately lean:

- **Cím** — the H1: `# <story title> · STORY-…`. The parent link is recorded as a `Parent epic: \`EPIC-…\`` line (or `Parent epic: _(nincs — epic nélküli story)_` in epic-less mode), placed **inside the pipeline-only fence** (not under the H1), so it never reaches the Jira Description — the parent is carried by Jira's structured `parent` field instead.
- **`## User Story`** — opens with the `Mint [szerep], szeretnék [cél], hogy [érték]` line, **reused verbatim** from the epic (only sharpen it if the epic's line is a placeholder, and note when you do), then one or two sentences of narrative. There is **no** separate *User story* heading — the user-story line lives here.
- **`## Context`** — one or two sentences: the value slice this story delivers, carried from the epic — never a technical layer.
- **`## BDD Test`** — a placeholder for the declarative Gherkin that `pte-openspec-bdd-tests` fills in place from the story's mapped `#### Scenario`s. **This Gherkin is the story's acceptance criteria** — the new schema drops the separate *Elfogadási kritériumok* section; the `#### Scenario` `WHEN`/`THEN` become the `Adott/Amikor/Akkor` steps of the test, not a duplicated prose list. Author this section as a marker only — do **not** write Gherkin here:

  ```markdown
  ## BDD Test
  _(kitölti a `pte-openspec-bdd-tests`)_
  ```
- **`## Risks and Dependencies`** — dependencies + risks, including sibling stories within the same epic (replaces *Függőségek és kockázatok*). This is the **last Jira-visible section**.
- **`## Estimation`** — the story's **reference base** estimate, placed **inside a `<!-- pipeline-only -->` fence at the end of the card** (so it is stripped from the Jira Description). The **rendered** section is three stacked bullets: the **`3-Points becslés`** (the PERT three points `O`/`M`/`P`, the expected `Eβ`, and `σ`, **inline on one line** — `O <O> / M <M> / P <P> | **Eβ <Eβ>**, σ <σ>`, a `|` before the **bolded** `Eβ`), then **`Becsült munkaóra:`** (`Eβ` ideal engineer-hours + the ideal-day equivalent), then **`Story Point:`** (`<SP>`, a bare integer) — plus the `⚠` split-warning line when it trips. **Do not guess the hours** — derive `O`/`M`/`P` by scoring the **weighted complexity rubric** (M-drivers + modifiers → `M`; σ-drivers → the O/P spread) in [`../pte-openspec-shared/ESTIMATION.md`](../pte-openspec-shared/ESTIMATION.md), the single source of truth for the rubric, PERT formula, Eβ→SP table, 6h-day anchor, split-warning threshold, and exact rendered format. Although fenced (hidden from the Description), `pte-openspec-jira-sync` **still pushes** its `SP` → *Story points* and `Becsült munkaóra`/`Eβ` → *Original estimate*; the multiplier layer stays downstream and out of scope.
- **`## Dependency Edges`** — the card's **outbound edges**, machine-readable per the [*Dependency Edges* contract in `CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md#the-dependency-edges-block--the-edge-contract), placed **inside the same `<!-- pipeline-only -->` fence** as `## Estimation`. One tagged bullet per edge: `depends-on:`/`blocks:`/`relates-to:` + a backticked `EPIC-…`/`STORY-…` (including sibling stories in the same epic), or `external:` + free text. **Derive it best-effort** from the `## Risks and Dependencies` narrative you just wrote and the story's mapped scenarios — a sibling or system named there as a dependency becomes an edge. A human refines it. Single source of truth for the dependency graph and the Jira links; the `## Risks and Dependencies` prose is never parsed for edges.

**Dropped from the old schema** (do not emit them): the *Elfogadási kritériumok* section (now the BDD Gherkin), the *INVEST-ellenőrzés* section, *Készenléti feltétel (DoR)* / *Elkészültségi feltétel (DoD)*, *Prioritás*, and — for epic-driven stories — the *Source Reference* trace row. (The old *Becslés* section is **not** dropped — it is **reintroduced** as `## Estimation` above, reshaped into the PERT + Story-Points reference base.) **Note:** only the printed *INVEST-ellenőrzés* section is dropped — INVEST itself is **not** dropped; it is now a mandatory silent rule.

**INVEST is a rule, not a section — every story MUST satisfy it** (all six letters), enforced as a hard silent gate per [*INVEST — the story gate* in `CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md#invest--the-story-gate): a card that cannot be made INVEST-conform is re-cut, not shipped — epic-driven, kick the re-split back to `pte-openspec-to-epics`; epic-less, reshape here.

**Trace lives in the epic, not the card.** An epic-driven card carries **no** trace row: `pte-openspec-to-stories` and `pte-openspec-bdd-tests` read the epic's (pipeline-only) *Source Spec Reference* map as the single source of the requirement→story→scenario split; the card's `STORY-…`/`EPIC-…` IDs live only in its **Cím** and filename. Every card **does** carry a pipeline-only fence — holding `## Estimation` and `## Dependency Edges` (epic-driven), and in **epic-less mode** (below) the card also self-carries its *Source Reference* map row inside that same fence.

## Trace IDs

Per [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md), in its normal (epic-driven) mode this skill **CONSUMES** the IDs `pte-openspec-to-epics` minted — it reuses each `STORY-…` and its parent `EPIC-…` verbatim and **never mints**. Produce **exactly one card per `STORY-…` row** in the epic map — no more, no fewer.

## Epic-less mode — a co-equal path for small standalone changes

Not every change deserves an epic. When a **change request warrants a story but not an epic** (a small, single-capability change with no strategic decomposition to do), run **epic-less**: author the story card(s) straight from the change's delta spec, no epic in between.

This is a **first-class mode, not a degraded fallback** — use it deliberately for right-sized small changes; the only thing that differs from epic-driven mode is where the IDs and the requirement→story split come from:

- **Mint the IDs here.** Per `CONVENTIONS.md`, use `STORY-<capability-slug>-<requirement-slug>`, namespaced on the capability (from `openspec/specs/<capability>/`). These are **stable, not provisional** — the card leaves the **parent `EPIC-…` line empty** (a marker the story is epic-less), and `pte-openspec-jira-sync` parents it under the standalone collector epic.
- **Do the split yourself.** With no epic map to consume, read the delta's `### Requirement` / `#### Scenario` blocks and cut vertical-slice stories by user value — the same judgement `pte-openspec-to-epics` applies, scoped to this one change. Keep it small: if you find yourself minting many stories or wanting real strategic framing, that's the signal to stop and run `pte-openspec-to-epics` (mint or reconcile) instead.
- **Self-carry the map.** With no epic to hold the *Source Spec Reference* table, an epic-less card carries its **own** trace row inside a pipeline-only fence, so `pte-openspec-bdd-tests` can read which `#### Scenario`s the card covers (hidden from Jira, same fence as the epic's map):

  ````markdown
  <!-- pipeline-only:start -->
  ## Source Reference
  | Story ID | Epic ID | Source `### Requirement` | Covered `#### Scenario`s |
  |----------|---------|--------------------------|----------------------------|
  <!-- pipeline-only:end -->
  ````

  (Epic-driven cards do **not** get this block — their map lives in the epic.)

Everything else — the visible card anatomy, the 1:1 Scenario→BDD trace, the `BDD Test` placeholder, diff-don't-clobber — is identical to epic-driven mode.

**When the change touches a capability that already has an epic, this is the wrong mode:** don't orphan the story — run `pte-openspec-to-epics` in **reconcile** mode to attach it under the affected epic, then expand it here normally.

## Steps

Copy this checklist and tick each item — the verify step is exhaustive, not a glance:

```
- [ ] 1. Mode fixed: source epic file set named (epic-driven) or epic-less mode declared
- [ ] 2. Every STORY-… row + its mapped #### Scenario-k enumerated from the epic map
- [ ] 3. Mapped Scenarios traced to the spec for acceptance-criteria detail
- [ ] 4. One story card authored per STORY-… row (full anatomy incl. ## Estimation + ## Dependency Edges; STORY-…/EPIC-… reused verbatim; every card INVEST-conform)
- [ ] 5. Files written (diffed, not clobbered)
- [ ] 6. Every STORY-… has exactly one card; every mapped Scenario covered; every card carries a PERT+SP ## Estimation (split-warning flagged where it trips); every card INVEST-conform; gaps reported
```

1. **Resolve the source epics — or declare epic-less.** Locate the epic files (default `openspec/backlog/epics/`, configurable — see *Output location* in `CONVENTIONS.md`). If several epics exist and it is unclear which to expand, list them with **AskUserQuestion**. If no epic exists for the target capability **and** the change is small enough not to warrant one, run **epic-less mode** (mint capability-namespaced IDs from the delta, per that section); if the capability *should* have an epic, stop and offer `pte-openspec-to-epics` (mint or reconcile) first. Completion: the mode is fixed — the exact epic file set is named, or epic-less mode is declared with its source delta spec.

2. **Enumerate the stories.** From each epic's *User Stories* + *Source Spec Reference* map, list every `STORY-…` ID, its `Mint …` line, and the `#### Scenario`s the map assigns to it. Completion (exhaustive): every `STORY-…` row in every source epic is on the list; none invented, none dropped.

3. **Trace acceptance criteria to the spec.** For each story, open the source `### Requirement` / `#### Scenario`s named in its map row (resolve the spec per `CONVENTIONS.md`) and read their `WHEN`/`THEN` bullets — the behavioural source that `pte-openspec-bdd-tests` will turn into the card's BDD test. Completion: every mapped Scenario's `WHEN`/`THEN` is in hand for its story.

4. **Author one card per story — INVEST-conform.** Write the lean anatomy above: reuse the `STORY-…`/`EPIC-…` IDs verbatim (Cím + parent line), open `## User Story` with the epic's `Mint …` line verbatim, write `## Context`, leave the `## BDD Test` placeholder empty (`pte-openspec-bdd-tests` fills it — that Gherkin is the acceptance criteria), write the `## Estimation` reference base — score the weighted complexity rubric to derive `O`/`M`/`P` (M-drivers + modifiers → `M`; σ-drivers → the O/P spread), then `Eβ`, `σ`, the Eβ→SP derivation, and the `⚠` split-warning where it trips — all per `ESTIMATION.md`, write `## Risks and Dependencies`, and — inside the same pipeline-only fence — the `## Dependency Edges` block (derived best-effort from that narrative + the mapped scenarios; sibling/system dependencies become tagged edges). **Do not** emit the dropped sections (Elfogadási kritériumok / INVEST section / DoR / DoD / Prioritás / — for epic-driven — Source Reference). In epic-less mode, also emit the self-carried pipeline-only *Source Reference* block. **Every card must pass the INVEST gate** (see the rule above) — check all six letters before writing; if a story fails one, reshape/split it (epic-driven: kick back to `pte-openspec-to-epics`) rather than emit a non-conforming card. Completion: every anatomy section present (including the empty `BDD Test` placeholder); no dropped section emitted; every card INVEST-conform; no behaviour invented.

5. **Write the files.** Default `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md` (epic-less: `<capability-slug>` in place of `<epic-slug>`), one card per file — the **filename is the card's full `STORY-…` trace ID** + `.md` (searchability parity with the `EPIC-…` files; see *Output location* in `CONVENTIONS.md`). Diff before overwriting — never clobber hand-edited content. Completion: each story exists as its own file under the output dir.

6. **Verify exhaustively.** Every `STORY-…` in every source epic has exactly one card; every card has the five visible sections (Description / Context / BDD Test placeholder / Estimation / Risks and Dependencies) and no dropped section; every `## Estimation` carries a PERT base + derived SP (and the `⚠` split-warning wherever `SP ≥ 13` or `σ/Eβ > 0.5`); **every card satisfies INVEST** (all six letters — flag any story that does not, and reshape/split rather than ship it); every reused ID matches the epic verbatim (Cím + parent line); epic-less cards carry the self-map inside the pipeline-only fence and epic-driven cards do not. Report any story or Scenario you could not place cleanly rather than guessing.
