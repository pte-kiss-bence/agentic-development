---
name: pte-openspec-to-stories
description: Expand an OpenSpec epic's stories into detailed, refinement-ready Agile story cards. Use when the user wants story cards or a groomed/refined backlog from an epic produced by pte-openspec-to-epics, wants epic user stories fleshed out with acceptance criteria and INVEST checks, or mentions turning OpenSpec stories into ready-for-sprint cards.
---

An Agile epic from `pte-openspec-to-epics` already lists its stories — each a `STORY-…` ID, a `Mint … szeretnék … hogy …` line, and high-level acceptance criteria in its *User story-k* section. This skill expands **each** of those stories into a full, **refinement-ready** Agile story card — one markdown file per story.

`refinement-ready` is the governing word: a card is what a team can pull into planning — a clear user-value statement, testable acceptance criteria, an INVEST check, and the placeholders (priority, estimate, Definition of Ready/Done) a team fills during refinement. It never invents behaviour: acceptance criteria are expanded 1:1 from the `#### Scenario`s the epic's map assigns to the story, traced back to the spec.

Shared pipeline conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, and diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md); read it before writing. A complete worked transformation is in [`EXAMPLE.md`](EXAMPLE.md).

## Role in the pipeline

The **primary input is the epic file(s)** produced by `pte-openspec-to-epics`, not the raw spec. The epic owns the story set and the IDs; this skill only expands each story. The spec is the **traced-through source** for the acceptance-criteria detail: for each story, follow the epic's *Forrás-spec hivatkozás* map to the exact `### Requirement` / `#### Scenario` blocks and expand their `WHEN`/`THEN`.

A card embodies the **3 C's**: it is the *Card*, it stands in for a *Conversation*, and its *Elfogadási kritériumok* are the *Confirmation*.

## Source mapping

| Agile Epic (`pte-openspec-to-epics`) | Story card |
|--------------------------------------|-----------|
| one *User story-k* entry (`STORY-…` + `Mint …` line) | one story card file |
| the story's `STORY-…` ID + parent `EPIC-…` | **Cím** trace header (verbatim) |
| the epic's `Mint … szeretnék … hogy …` line | **User story** (verbatim) |
| the `#### Scenario`s the *Forrás-spec hivatkozás* row assigns, traced to the spec's `WHEN`/`THEN` | **Elfogadási kritériumok** (`Amennyiben`/`Amikor`/`Akkor`) |
| the epic's *Forrás-spec hivatkozás* row | **Forrás-hivatkozás** trace row |

## Story card anatomy

Each card carries these sections, in order (Hungarian headings — use as-is):

- **Cím** — story title, its `STORY-…` ID, and parent `EPIC-…`, both taken **verbatim** from the epic.
- **User story** — `Mint [szerep], szeretnék [cél], hogy [érték]`, **reused verbatim** from the epic; only sharpen the wording if the epic's line is a placeholder, and note when you do.
- **Kontextus** — one or two sentences: the value slice this story delivers, carried from the epic — never a technical layer.
- **Elfogadási kritériumok** — `Amennyiben` / `Amikor` / `Akkor` (`És` for extra outcomes), expanded **1:1** from the `#### Scenario`s the epic map assigns to this story. Supply the `Amennyiben` precondition the `WHEN` assumes (the spec states trigger + outcome, not the precondition). Never invent a criterion, and never pull one from a Scenario the map did not assign.
- **INVEST-ellenőrzés** — one line confirming Independent, Negotiable, Valuable, Estimable, Small, Testable; flag any letter it fails.
- **Készenléti feltétel (DoR)** / **Elkészültségi feltétel (DoD)** — placeholders for the team.
- **Prioritás** — placeholder.
- **Becslés** — story-point / relative-size placeholder, left blank for refinement (never guessed).
- **Függőségek és kockázatok** — including sibling stories within the same epic.
- **Forrás-hivatkozás** — the trace row: `STORY-…`, parent `EPIC-…`, source `### Requirement`, covered `#### Scenario`s (titles verbatim).

## Trace IDs

Per [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md), this skill **CONSUMES** the IDs `pte-openspec-to-epics` minted — it reuses each `STORY-…` and its parent `EPIC-…` verbatim and **never mints**. Produce **exactly one card per `STORY-…` row** in the epic map — no more, no fewer.

**Fallback only when no epic exists:** if the target spec has no epic yet, say so and offer to run `pte-openspec-to-epics` first (preferred). Only if the user insists, derive provisional `STORY-<capability>-<requirement-slug>` IDs straight from the spec, clearly flagged as provisional until an epic reconciles them. This is a degraded path, not a co-equal mode.

## Steps

Copy this checklist and tick each item — the verify step is exhaustive, not a glance:

```
- [ ] 1. Source epic file set named (or no-epic fallback flagged)
- [ ] 2. Every STORY-… row + its mapped #### Scenario-k enumerated from the epic map
- [ ] 3. Mapped Scenarios traced to the spec for acceptance-criteria detail
- [ ] 4. One story card authored per STORY-… row (full anatomy; STORY-…/EPIC-… reused verbatim)
- [ ] 5. Files written (diffed, not clobbered)
- [ ] 6. Every STORY-… has exactly one card; every mapped Scenario covered by its card's AC; gaps reported
```

1. **Resolve the source epics.** Locate the epic files (default `epics/`, configurable). If several epics exist and it is unclear which to expand, list them with **AskUserQuestion**. If no epic exists for the target spec, follow the no-epic fallback above. Completion: the exact epic file set (or the flagged fallback) is named.

2. **Enumerate the stories.** From each epic's *User story-k* + *Forrás-spec hivatkozás* map, list every `STORY-…` ID, its `Mint …` line, and the `#### Scenario`s the map assigns to it. Completion (exhaustive): every `STORY-…` row in every source epic is on the list; none invented, none dropped.

3. **Trace acceptance criteria to the spec.** For each story, open the source `### Requirement` / `#### Scenario`s named in its map row (resolve the spec per `CONVENTIONS.md`) and read their `WHEN`/`THEN` bullets — the behavioural source for the card's AC. Completion: every mapped Scenario's `WHEN`/`THEN` is in hand for its story.

4. **Author one card per story.** Write the full anatomy above: reuse the `STORY-…`/`EPIC-…` IDs and `Mint …` line verbatim, expand each mapped Scenario's `WHEN`/`THEN` into `Amennyiben`/`Amikor`/`Akkor` acceptance criteria (supplying the precondition), add the INVEST check, and leave priority/estimate/DoR/DoD as team placeholders. Completion: every anatomy section present; AC trace 1:1 to the mapped Scenarios; no behaviour invented.

5. **Write the files.** Default `stories/<epic-slug>/<story-slug>.md`, one card per file, slugged from the `STORY-…` ID or title. Diff before overwriting — never clobber hand-edited content. Completion: each story exists as its own file under the output dir.

6. **Verify exhaustively.** Every `STORY-…` in every source epic has exactly one card; every `#### Scenario` the map assigned appears in its card's acceptance criteria; every reused ID matches the epic verbatim. Report any story or Scenario you could not place cleanly rather than guessing.
