---
name: pte-openspec-to-stories
description: Expand an OpenSpec epic's stories into detailed, refinement-ready Agile story cards — or author standalone cards straight from a small change's delta spec (epic-less mode). Use when the user wants story cards or a groomed/refined backlog from an epic produced by pte-openspec-to-epics, wants epic user stories fleshed out into lean refinement-ready cards whose acceptance criteria are declarative BDD scenarios, mentions turning OpenSpec stories into ready-for-sprint cards, or wants a small change turned into one or a few story cards without creating an epic.
---

An Agile epic from `pte-openspec-to-epics` already lists its stories — each a `STORY-…` ID, a `Mint … szeretnék … hogy …` line, and high-level acceptance criteria in its *User story-k* section. This skill expands **each** of those stories into a full, **refinement-ready** Agile story card — one markdown file per story.

`refinement-ready` is the governing word: a card is what a team can pull into planning — a clear user-value statement (`## Description`), the value slice (`## Context`), testable acceptance criteria as declarative Gherkin (`## BDD Test`, filled by `pte-openspec-bdd-tests`), and its risks/dependencies. The schema is lean by design — it never invents behaviour: the acceptance criteria are the `#### Scenario`s the epic's map assigns to the story, traced back to the spec and expressed as the story's BDD test.

The story card is the pipeline's **smallest unit and the home of its own BDD test(s)**: the card carries a `BDD Test` section that `pte-openspec-bdd-tests` fills in place with declarative Gherkin. This skill authors that section as an explicit placeholder — it does **not** write Gherkin itself.

Shared pipeline conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, and diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md); read it before writing. A complete worked transformation is in [`EXAMPLE.md`](EXAMPLE.md).

## Role in the pipeline

The **primary input is the epic file(s)** produced by `pte-openspec-to-epics`, not the raw spec. The epic owns the story set and the IDs; this skill only expands each story. The spec is the **traced-through source** for the acceptance-criteria detail: for each story, follow the epic's *Forrás-spec hivatkozás* map to the exact `### Requirement` / `#### Scenario` blocks and expand their `WHEN`/`THEN`.

A card embodies the **3 C's**: it is the *Card*, it stands in for a *Conversation*, and its *BDD Test* (the Gherkin scenarios `pte-openspec-bdd-tests` fills) is the *Confirmation*.

## Source mapping

| Agile Epic (`pte-openspec-to-epics`) | Story card |
|--------------------------------------|-----------|
| one *User story-k* entry (`STORY-…` + `Mint …` line) | one story card file |
| the story's `STORY-…` ID + parent `EPIC-…` | **Cím** trace header (verbatim) |
| the epic's `Mint … szeretnék … hogy …` line | first line of **`## Description`** (verbatim) |
| the `#### Scenario`s the *Forrás-spec hivatkozás* row assigns | the **`## BDD Test`** placeholder (Gherkin filled later by `pte-openspec-bdd-tests` — that Gherkin **is** the acceptance criteria) |
| the epic's *Forrás-spec hivatkozás* row | **stays in the epic** — the card carries no trace row (epic-less: self-carried, pipeline-only) |

## Story card anatomy

Section **headings are English** (the new schema — see the golden sample in [`EXAMPLE.md`](EXAMPLE.md)); the **content stays Hungarian** (per `CONVENTIONS.md`). Each card carries these sections, in order — the schema is deliberately lean:

- **Cím** — the H1: `# <story title> · STORY-…`, with the parent `EPIC-…` noted verbatim from the epic (empty in epic-less mode).
- **`## Description`** — opens with the `Mint [szerep], szeretnék [cél], hogy [érték]` line, **reused verbatim** from the epic (only sharpen it if the epic's line is a placeholder, and note when you do), then one or two sentences of narrative. There is **no** separate *User story* heading — the user-story line lives here.
- **`## Context`** — one or two sentences: the value slice this story delivers, carried from the epic — never a technical layer.
- **`## BDD Test`** — a placeholder for the declarative Gherkin that `pte-openspec-bdd-tests` fills in place from the story's mapped `#### Scenario`s. **This Gherkin is the story's acceptance criteria** — the new schema drops the separate *Elfogadási kritériumok* section; the `#### Scenario` `WHEN`/`THEN` become the `Adott/Amikor/Akkor` steps of the test, not a duplicated prose list. Author this section as a marker only — do **not** write Gherkin here:

  ```markdown
  ## BDD Test
  _(kitölti a `pte-openspec-bdd-tests`)_
  ```
- **`## Risks and Dependencies`** — dependencies + risks, including sibling stories within the same epic (replaces *Függőségek és kockázatok*).

**Dropped from the old schema** (do not emit them): the *Elfogadási kritériumok* section (now the BDD Gherkin), the *INVEST-ellenőrzés* section, *Készenléti feltétel (DoR)* / *Elkészültségi feltétel (DoD)*, *Prioritás*, *Becslés*, and — for epic-driven stories — the *Forrás-hivatkozás* trace row. **Note:** only the printed *INVEST-ellenőrzés* section is dropped — INVEST itself is **not** dropped; it is now a mandatory rule (below), enforced without a section.

**INVEST is a rule, not a section — every story MUST satisfy it.** The lean card no longer prints an *INVEST-ellenőrzés* block, but every story card this skill authors **must** conform to INVEST — **I**ndependent, **N**egotiable, **V**aluable, **E**stimable, **S**mall, **T**estable. Check each story against all six letters while authoring; if a story fails one (e.g. not **I**ndependent, or too big to be **S**mall/**E**stimable), fix it — sharpen the value slice, or split it into sibling stories — rather than emitting a non-conforming card. The conformance is silent (no section), but it is a hard gate: a card that cannot be made INVEST-conform is a signal to reshape the split, not to ship. In epic-driven mode a story that only becomes INVEST-conform by re-splitting is a signal to go back to `pte-openspec-to-epics` (the epic owns the split); in epic-less mode, do the reshape here.

**Trace lives in the epic, not the card.** An epic-driven card carries **no** trace row: `pte-openspec-to-stories` and `pte-openspec-bdd-tests` read the epic's (pipeline-only) *Forrás-spec hivatkozás* map as the single source of the requirement→story→scenario split; the card's `STORY-…`/`EPIC-…` IDs live only in its **Cím** and filename. The one exception is **epic-less mode** (below), where the card self-carries its map inside a pipeline-only fence.

## Trace IDs

Per [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md), in its normal (epic-driven) mode this skill **CONSUMES** the IDs `pte-openspec-to-epics` minted — it reuses each `STORY-…` and its parent `EPIC-…` verbatim and **never mints**. Produce **exactly one card per `STORY-…` row** in the epic map — no more, no fewer.

## Epic-less mode — a co-equal path for small standalone changes

Not every change deserves an epic. When a **change request warrants a story but not an epic** (a small, single-capability change with no strategic decomposition to do), run **epic-less**: author the story card(s) straight from the change's delta spec, no epic in between.

This is a **first-class mode, not a degraded fallback** — use it deliberately for right-sized small changes; the only thing that differs from epic-driven mode is where the IDs and the requirement→story split come from:

- **Mint the IDs here.** Per `CONVENTIONS.md`, use `STORY-<capability-slug>-<requirement-slug>`, namespaced on the capability (from `openspec/specs/<capability>/`). These are **stable, not provisional** — the card leaves the **parent `EPIC-…` line empty** (a marker the story is epic-less), and `pte-openspec-jira-sync` parents it under the standalone collector epic.
- **Do the split yourself.** With no epic map to consume, read the delta's `### Requirement` / `#### Scenario` blocks and cut vertical-slice stories by user value — the same judgement `pte-openspec-to-epics` applies, scoped to this one change. Keep it small: if you find yourself minting many stories or wanting real strategic framing, that's the signal to stop and run `pte-openspec-to-epics` (mint or reconcile) instead.
- **Self-carry the map.** With no epic to hold the *Forrás-spec hivatkozás* table, an epic-less card carries its **own** trace row inside a pipeline-only fence, so `pte-openspec-bdd-tests` can read which `#### Scenario`s the card covers (hidden from Jira, same fence as the epic's map):

  ````markdown
  <!-- pipeline-only:start -->
  ## Forrás-hivatkozás
  | Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
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
- [ ] 4. One story card authored per STORY-… row (full anatomy; STORY-…/EPIC-… reused verbatim; every card INVEST-conform)
- [ ] 5. Files written (diffed, not clobbered)
- [ ] 6. Every STORY-… has exactly one card; every mapped Scenario covered; every card INVEST-conform; gaps reported
```

1. **Resolve the source epics — or declare epic-less.** Locate the epic files (default `openspec/backlog/epics/`, configurable — see *Output location* in `CONVENTIONS.md`). If several epics exist and it is unclear which to expand, list them with **AskUserQuestion**. If no epic exists for the target capability **and** the change is small enough not to warrant one, run **epic-less mode** (mint capability-namespaced IDs from the delta, per that section); if the capability *should* have an epic, stop and offer `pte-openspec-to-epics` (mint or reconcile) first. Completion: the mode is fixed — the exact epic file set is named, or epic-less mode is declared with its source delta spec.

2. **Enumerate the stories.** From each epic's *User story-k* + *Forrás-spec hivatkozás* map, list every `STORY-…` ID, its `Mint …` line, and the `#### Scenario`s the map assigns to it. Completion (exhaustive): every `STORY-…` row in every source epic is on the list; none invented, none dropped.

3. **Trace acceptance criteria to the spec.** For each story, open the source `### Requirement` / `#### Scenario`s named in its map row (resolve the spec per `CONVENTIONS.md`) and read their `WHEN`/`THEN` bullets — the behavioural source that `pte-openspec-bdd-tests` will turn into the card's BDD test. Completion: every mapped Scenario's `WHEN`/`THEN` is in hand for its story.

4. **Author one card per story — INVEST-conform.** Write the lean anatomy above: reuse the `STORY-…`/`EPIC-…` IDs verbatim (Cím + parent line), open `## Description` with the epic's `Mint …` line verbatim, write `## Context`, leave the `## BDD Test` placeholder empty (`pte-openspec-bdd-tests` fills it — that Gherkin is the acceptance criteria), and write `## Risks and Dependencies`. **Do not** emit the dropped sections (Elfogadási kritériumok / INVEST section / DoR / DoD / Prioritás / Becslés / — for epic-driven — Forrás-hivatkozás). In epic-less mode, also emit the self-carried pipeline-only *Forrás-hivatkozás* block. **Every card must pass the INVEST gate** (see the rule above) — check all six letters before writing; if a story fails one, reshape/split it (epic-driven: kick back to `pte-openspec-to-epics`) rather than emit a non-conforming card. Completion: every anatomy section present (including the empty `BDD Test` placeholder); no dropped section emitted; every card INVEST-conform; no behaviour invented.

5. **Write the files.** Default `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md` (epic-less: `<capability-slug>` in place of `<epic-slug>`), one card per file — the **filename is the card's full `STORY-…` trace ID** + `.md` (searchability parity with the `EPIC-…` files; see *Output location* in `CONVENTIONS.md`). Diff before overwriting — never clobber hand-edited content. Completion: each story exists as its own file under the output dir.

6. **Verify exhaustively.** Every `STORY-…` in every source epic has exactly one card; every card has the four visible sections (Description / Context / BDD Test placeholder / Risks and Dependencies) and no dropped section; **every card satisfies INVEST** (all six letters — flag any story that does not, and reshape/split rather than ship it); every reused ID matches the epic verbatim (Cím + parent line); epic-less cards carry the self-map inside the pipeline-only fence and epic-driven cards do not. Report any story or Scenario you could not place cleanly rather than guessing.
