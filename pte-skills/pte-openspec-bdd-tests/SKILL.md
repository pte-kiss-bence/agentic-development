---
name: pte-openspec-bdd-tests
description: Generate declarative BDD test scenarios (Gherkin) and embed them into the story cards' BDD teszt section. Use when the user wants BDD, Gherkin, or Cucumber scenarios for story cards produced by pte-openspec-to-stories, mentions "BDD tests from specs/stories", or wants spec scenarios turned into living documentation inside the story cards (later implemented as Playwright E2E tests).
---

An OpenSpec spec already carries behaviour: each `### Requirement` holds `#### Scenario` blocks of `WHEN`/`THEN` bullets. That is BDD in embryo. This skill turns it into **declarative** Gherkin and writes it **into the story cards** — each card owns its own BDD test(s), so the story is a self-contained, testable unit. The Gherkin is **living documentation** — human-readable, not executable in this pipeline; a **later, separate phase implements it as a Playwright E2E test**. It stays stable when the implementation changes, never leaking UI or procedural detail (the UI/Playwright detail lives in that future E2E layer, never in the Gherkin).

This is the **last step of the planning chain** (`epics` → `stories` → **bdd-tests**; the build stage `pte-openspec-tdd-apply` is a separate phase). It produces no separate `.feature` tree and no step-definition files: it fills the `BDD teszt` section that `pte-openspec-to-stories` left as a placeholder in each card, and touches nothing else.

`declarative` is the governing word: every generated step describes _what_ the system does, not _how_ a user clicks it. The full rule set and anti-patterns live in [`BDD-RULES.md`](BDD-RULES.md) — load it before writing any step. A complete worked transformation is in [`EXAMPLE.md`](EXAMPLE.md).

Shared pipeline conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, and diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md).

## Role in the pipeline

The **primary input is the story card set** produced by `pte-openspec-to-stories`, not the raw spec. Each card names, in its *Forrás-hivatkozás* row, its `STORY-…`/`EPIC-…` IDs, its source `### Requirement`, and the `#### Scenario`s it covers. This skill:

1. reads that row to learn **which** Scenarios belong to the card and **which** trace IDs to tag with, then
2. traces those `#### Scenario`s back to the spec for the **behavioural source** (`WHEN`/`THEN`) — Gherkin steps always come from the spec, never from the card's prose acceptance criteria, and
3. writes the Gherkin into the card's `BDD teszt` section, in place.

One card → one BDD block, scoped to exactly the Scenarios that card's map row assigns. A Scenario the row does not assign never appears in that card.

## Source mapping

OpenSpec and Gherkin line up almost one-to-one. The single gap is `Given`: OpenSpec scenarios state the trigger and outcome but rarely the precondition, so you must supply it. Because the output is scoped to one story, the block carries that story's trace IDs as tags.

| Story card / OpenSpec | Gherkin (embedded in the card's `BDD teszt`) |
|-----------------------|----------------------------------------------|
| the card's `EPIC-…` / `STORY-…` IDs | `@EPIC-<epic-slug>` / `@STORY-…` tags on the block |
| the card's source `### Requirement` (its SHALL text) | `Rule:` (with a `Feature:` for the capability) |
| a covered `#### Scenario: <name>` | `Scenario:` — or `Scenario Outline:` when only data varies |
| (precondition implied by the Requirement / WHEN) | `Given …` — **added**, not present in the source |
| `- **WHEN** …` | `When …` (one event; extra bullets become `And` only if genuinely separate events) |
| `- **THEN** …` | `Then …` (extra bullets become `And`) |

## Where the Gherkin goes

Replace the card's placeholder with a fenced `gherkin` block. Hungarian spec ⇒ `# language: hu` as the block's first line and Hungarian Gherkin keywords (`Jellemző`/`Szabály`/`Háttér`/`Forgatókönyv`/`Adott`/`Amikor`/`Akkor`/`És`/`Példák`):

```markdown
## BDD teszt
```gherkin
# language: hu
# Spec: <capability> › <Requirement name>
@EPIC-<epic-slug> @STORY-<epic-slug>-<requirement-slug>
Jellemző: <capability>
  Szabály: <Requirement name>
    Forgatókönyv: <Scenario name>
      Adott …
      Amikor …
      Akkor …
```
```

## Steps

Copy this checklist and tick each item as you go — the verify step is exhaustive, not a glance:

```
- [ ] 1. Source story card set named (fallback flagged if no cards exist)
- [ ] 2. Every card's STORY-… + its mapped #### Scenario-k + trace IDs enumerated
- [ ] 3. Mapped Scenarios traced to the spec for WHEN/THEN
- [ ] 4. Declarative Gherkin written per card against BDD-RULES.md
- [ ] 5. Trace comment + @EPIC-…/@STORY-… tags added from the card's row
- [ ] 6. Each card's BDD teszt section filled in place (diffed, not clobbered; nothing else touched)
- [ ] 7. Every card has Gherkin; every mapped Scenario present; unmappable bullets reported
```

1. **Resolve the source story cards.** Locate the cards (default `stories/`, configurable). If several epics' cards exist and it is unclear which to cover, list them with **AskUserQuestion**. **When no cards exist:** stop and offer to run `pte-openspec-to-stories` first — that is the pipeline's correct entry into this stage. There is no standalone `.feature` path: this skill only fills the `BDD teszt` section of existing cards and writes no new files. Completion: the exact card set is named (or the skill has stopped and offered `pte-openspec-to-stories`).

2. **Enumerate per card.** From each card's *Forrás-hivatkozás* row, list its `STORY-…`/`EPIC-…` IDs, its source `### Requirement`, and the `#### Scenario`s it covers. Completion (exhaustive): every card and every Scenario its row assigns is on the list; none invented, none dropped.

3. **Trace to the spec.** For each covered `#### Scenario`, open its `### Requirement` in the spec (resolve per `CONVENTIONS.md`) and read the `WHEN`/`THEN` bullets — the behavioural source. Completion: every mapped Scenario's `WHEN`/`THEN` is in hand.

4. **Write declarative Gherkin per card** against [`BDD-RULES.md`](BDD-RULES.md):
   - Supply the missing `Given` — the state the `WHEN` assumes.
   - `WHEN` → a single `When`. Multiple `WHEN` bullets are a conjunction smell: keep one event, or split the Scenario if it tests two behaviours.
   - `THEN` → `Then` (+ `And` per extra outcome).
   - Collapse Scenarios that differ only in data into a single `Scenario Outline:` with an `Examples:` table; lift a `Given` shared by all of the card's Scenarios into a `Background:`.
   - Third-person business language in the **spec's own language**; emit `# language: <code>` as the block's first line when not English (`# language: hu` for Hungarian specs).
   - No UI, route, or field-level detail. Declarative, not procedural. Steps sourced from the spec's `WHEN`/`THEN`, never from the card's prose.

5. **Add traceability.** Comment the block with its origin (`# Spec: <capability> › <Requirement name>`; append the change id when sourced from a delta) and tag it with the card's own `@EPIC-<epic-slug>` and `@STORY-…` from the *Forrás-hivatkozás* row — verbatim, never re-minted (see `CONVENTIONS.md`; this skill CONSUMES the IDs).

6. **Fill the card in place.** Replace the `BDD teszt` placeholder in each story card with the fenced `gherkin` block. Diff before overwriting — never clobber hand-edited steps, and change **only** the `BDD teszt` section, leaving every other section of the card untouched. Write no separate files. Completion: every targeted card's `BDD teszt` section holds its Gherkin.

7. **Verify exhaustively.** Every source card has a filled `BDD teszt` section; every `#### Scenario` its map row assigned maps to a Gherkin `Scenario`/`Scenario Outline` in that card; every generated Scenario passes the [`BDD-RULES.md`](BDD-RULES.md) checklist; every `@EPIC-…`/`@STORY-…` tag matches the card's row verbatim; nothing appears that the spec does not state. Report any bullet you could not map cleanly (e.g. a `THEN` whose precondition the spec never gives) rather than guessing.
