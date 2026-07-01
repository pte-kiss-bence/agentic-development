---
name: pte-openspec-to-stories
description: Expand an OpenSpec epic's stories into detailed, refinement-ready Agile story cards — or author standalone cards straight from a small change's delta spec (epic-less mode). Use when the user wants story cards or a groomed/refined backlog from an epic produced by pte-openspec-to-epics, wants epic user stories fleshed out with acceptance criteria and INVEST checks, mentions turning OpenSpec stories into ready-for-sprint cards, or wants a small change turned into one or a few story cards without creating an epic.
---

An Agile epic from `pte-openspec-to-epics` already lists its stories — each a `STORY-…` ID, a `Mint … szeretnék … hogy …` line, and high-level acceptance criteria in its *User story-k* section. This skill expands **each** of those stories into a full, **refinement-ready** Agile story card — one markdown file per story.

`refinement-ready` is the governing word: a card is what a team can pull into planning — a clear user-value statement, testable acceptance criteria, an INVEST check, and the placeholders (priority, estimate, Definition of Ready/Done) a team fills during refinement. It never invents behaviour: acceptance criteria are expanded 1:1 from the `#### Scenario`s the epic's map assigns to the story, traced back to the spec.

The story card is the pipeline's **smallest unit and the home of its own BDD test(s)**: the card carries a `BDD teszt` section that `pte-openspec-bdd-tests` fills in place with declarative Gherkin. This skill authors that section as an explicit placeholder — it does **not** write Gherkin itself.

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
- **BDD teszt** — a placeholder for the declarative Gherkin that `pte-openspec-bdd-tests` fills in place from the same `#### Scenario`s. Author it as a marker only — do **not** write Gherkin here:

  ```markdown
  ## BDD teszt
  _(kitölti a `pte-openspec-bdd-tests`)_
  ```
- **INVEST-ellenőrzés** — one line confirming Independent, Negotiable, Valuable, Estimable, Small, Testable; flag any letter it fails.
- **Készenléti feltétel (DoR)** / **Elkészültségi feltétel (DoD)** — placeholders for the team.
- **Prioritás** — placeholder.
- **Becslés** — story-point / relative-size placeholder, left blank for refinement (never guessed).
- **Függőségek és kockázatok** — including sibling stories within the same epic.
- **Forrás-hivatkozás** — the trace row: `STORY-…`, parent `EPIC-…`, source `### Requirement`, covered `#### Scenario`s (titles verbatim).

## Trace IDs

Per [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md), in its normal (epic-driven) mode this skill **CONSUMES** the IDs `pte-openspec-to-epics` minted — it reuses each `STORY-…` and its parent `EPIC-…` verbatim and **never mints**. Produce **exactly one card per `STORY-…` row** in the epic map — no more, no fewer.

## Epic-less mode — a co-equal path for small standalone changes

Not every change deserves an epic. When a **change request warrants a story but not an epic** (a small, single-capability change with no strategic decomposition to do), run **epic-less**: author the story card(s) straight from the change's delta spec, no epic in between.

This is a **first-class mode, not a degraded fallback** — use it deliberately for right-sized small changes; the only thing that differs from epic-driven mode is where the IDs and the requirement→story split come from:

- **Mint the IDs here.** Per `CONVENTIONS.md`, use `STORY-<capability-slug>-<requirement-slug>`, namespaced on the capability (from `openspec/specs/<capability>/`). These are **stable, not provisional** — the card leaves the **parent `EPIC-…` line empty** (a marker the story is epic-less), and `pte-openspec-jira-sync` parents it under the standalone collector epic.
- **Do the split yourself.** With no epic map to consume, read the delta's `### Requirement` / `#### Scenario` blocks and cut vertical-slice stories by user value — the same judgement `pte-openspec-to-epics` applies, scoped to this one change. Keep it small: if you find yourself minting many stories or wanting real strategic framing, that's the signal to stop and run `pte-openspec-to-epics` (mint or reconcile) instead.

Everything else — the card anatomy, the 1:1 Scenario→AC trace, the `BDD teszt` placeholder, diff-don't-clobber — is identical to epic-driven mode.

**When the change touches a capability that already has an epic, this is the wrong mode:** don't orphan the story — run `pte-openspec-to-epics` in **reconcile** mode to attach it under the affected epic, then expand it here normally.

## Steps

Copy this checklist and tick each item — the verify step is exhaustive, not a glance:

```
- [ ] 1. Mode fixed: source epic file set named (epic-driven) or epic-less mode declared
- [ ] 2. Every STORY-… row + its mapped #### Scenario-k enumerated from the epic map
- [ ] 3. Mapped Scenarios traced to the spec for acceptance-criteria detail
- [ ] 4. One story card authored per STORY-… row (full anatomy; STORY-…/EPIC-… reused verbatim)
- [ ] 5. Files written (diffed, not clobbered)
- [ ] 6. Every STORY-… has exactly one card; every mapped Scenario covered by its card's AC; gaps reported
```

1. **Resolve the source epics — or declare epic-less.** Locate the epic files (default `openspec/backlog/epics/`, configurable — see *Output location* in `CONVENTIONS.md`). If several epics exist and it is unclear which to expand, list them with **AskUserQuestion**. If no epic exists for the target capability **and** the change is small enough not to warrant one, run **epic-less mode** (mint capability-namespaced IDs from the delta, per that section); if the capability *should* have an epic, stop and offer `pte-openspec-to-epics` (mint or reconcile) first. Completion: the mode is fixed — the exact epic file set is named, or epic-less mode is declared with its source delta spec.

2. **Enumerate the stories.** From each epic's *User story-k* + *Forrás-spec hivatkozás* map, list every `STORY-…` ID, its `Mint …` line, and the `#### Scenario`s the map assigns to it. Completion (exhaustive): every `STORY-…` row in every source epic is on the list; none invented, none dropped.

3. **Trace acceptance criteria to the spec.** For each story, open the source `### Requirement` / `#### Scenario`s named in its map row (resolve the spec per `CONVENTIONS.md`) and read their `WHEN`/`THEN` bullets — the behavioural source for the card's AC. Completion: every mapped Scenario's `WHEN`/`THEN` is in hand for its story.

4. **Author one card per story.** Write the full anatomy above: reuse the `STORY-…`/`EPIC-…` IDs and `Mint …` line verbatim, expand each mapped Scenario's `WHEN`/`THEN` into `Amennyiben`/`Amikor`/`Akkor` acceptance criteria (supplying the precondition), add the INVEST check, the empty `BDD teszt` placeholder, and leave priority/estimate/DoR/DoD as team placeholders. Completion: every anatomy section present (including the `BDD teszt` placeholder); AC trace 1:1 to the mapped Scenarios; no behaviour invented.

5. **Write the files.** Default `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md` (epic-less: `<capability-slug>` in place of `<epic-slug>`), one card per file — the **filename is the card's full `STORY-…` trace ID** + `.md` (searchability parity with the `EPIC-…` files; see *Output location* in `CONVENTIONS.md`). Diff before overwriting — never clobber hand-edited content. Completion: each story exists as its own file under the output dir.

6. **Verify exhaustively.** Every `STORY-…` in every source epic has exactly one card; every `#### Scenario` the map assigned appears in its card's acceptance criteria; every reused ID matches the epic verbatim. Report any story or Scenario you could not place cleanly rather than guessing.
