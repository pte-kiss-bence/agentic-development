---
name: pte-openspec-bdd-tests
description: Generate declarative BDD test scenarios (Gherkin/Cucumber) and embed them into the story cards' BDD Test section as living documentation. Use when the user wants Gherkin/BDD scenarios for the story cards produced by pte-openspec-to-stories (later implemented as Playwright E2E tests).
---

An OpenSpec spec already carries behaviour: each `### Requirement` holds `#### Scenario` blocks of `WHEN`/`THEN` bullets. That is BDD in embryo. This skill turns it into **declarative** Gherkin and writes it **into the story cards** — each card owns its own BDD test(s), so the story is a self-contained, testable unit. The Gherkin is **living documentation** — human-readable, not executable in this pipeline; a **later, separate phase implements it as a Playwright E2E test**. It stays stable when the implementation changes, never leaking UI or procedural detail (the UI/Playwright detail lives in that future E2E layer, never in the Gherkin).

This is the **last step of the planning chain** (`epics` → `stories` → **bdd-tests**; the build stage `pte-openspec-tdd-apply` is a separate phase). It produces no separate `.feature` tree and no step-definition files: it fills the `BDD Test` section that `pte-openspec-to-stories` left as a placeholder in each card, and touches nothing else.

`declarative` is the governing word: every generated step describes _what_ the system does, not _how_ a user clicks it. The full rule set and anti-patterns live in [`BDD-RULES.md`](BDD-RULES.md) — load it before writing any step. A complete worked transformation is in [`EXAMPLE.md`](EXAMPLE.md).

Shared pipeline conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, and diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md).

## Role in the pipeline

The **primary input is the story card set** produced by `pte-openspec-to-stories`, not the raw spec. In the new lean schema the story card carries **no** trace row of its own — the requirement→story→scenario split lives in the **epic's** pipeline-only *Forrás-spec hivatkozás* map. So the per-card scenario assignment and trace IDs come from **the map, matched to the card by its `STORY-…`** (from the card's *Cím* / filename):

- **Epic-driven card** → read the parent epic's (pipeline-only) *Forrás-spec hivatkozás* map; find the row whose `Story ID` matches this card. That row gives the `EPIC-…`/`STORY-…` tags, the source `### Requirement`, and the covered `#### Scenario`s.
- **Epic-less card** (empty parent `EPIC-…`) → the card **self-carries** its row inside its own pipeline-only fence; read it there.

This skill then:

1. reads that map row to learn **which** Scenarios belong to the card and **which** trace IDs to tag with, then
2. traces those `#### Scenario`s back to the spec for the **behavioural source** (`WHEN`/`THEN`) — Gherkin steps always come from the spec, never from card prose, and
3. writes the Gherkin into the card's `BDD Test` section, in place.

One card → one BDD block, scoped to exactly the Scenarios its map row assigns. A Scenario the row does not assign never appears in that card.

## Source mapping

OpenSpec and Gherkin line up almost one-to-one. The single gap is `Given`: OpenSpec scenarios state the trigger and outcome but rarely the precondition, so you must supply it. Because the output is scoped to one story, the block carries that story's trace IDs as tags.

| Story card / OpenSpec | Gherkin (embedded in the card's `BDD Test`) |
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
## BDD Test
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
- [ ] 5. Trace comment + @EPIC-…/@STORY-… tags added from the map row (epic map / epic-less self-row)
- [ ] 6. Each card's BDD Test section filled in place (diffed, not clobbered; nothing else touched)
- [ ] 7. Every card has Gherkin; every mapped Scenario present; unmappable bullets reported
```

1. **Resolve the source story cards.** Locate the cards (default `openspec/backlog/stories/`, configurable — see *Output location* in `CONVENTIONS.md`). If several epics' cards exist and it is unclear which to cover, list them with **AskUserQuestion**. **When no cards exist:** stop and offer to run `pte-openspec-to-stories` first — that is the pipeline's correct entry into this stage. There is no standalone `.feature` path: this skill only fills the `BDD Test` section of existing cards and writes no new files. Completion: the exact card set is named (or the skill has stopped and offered `pte-openspec-to-stories`).

2. **Enumerate per card — from the map, not the card.** For each card, find its map row: **epic-driven** → match the card's `STORY-…` against the parent epic's pipeline-only *Forrás-spec hivatkozás* table; **epic-less** → read the card's own pipeline-only *Forrás-hivatkozás* row. List the `STORY-…`/`EPIC-…` IDs, source `### Requirement`, and covered `#### Scenario`s. Completion (exhaustive): every card is matched to exactly one map row; every Scenario that row assigns is on the list; none invented, none dropped.

3. **Trace to the spec.** For each covered `#### Scenario`, open its `### Requirement` in the spec (resolve per `CONVENTIONS.md`) and read the `WHEN`/`THEN` bullets — the behavioural source. Completion: every mapped Scenario's `WHEN`/`THEN` is in hand.

4. **Write declarative Gherkin per card** against [`BDD-RULES.md`](BDD-RULES.md):
   - Supply the missing `Given` — the state the `WHEN` assumes.
   - `WHEN` → a single `When`. Multiple `WHEN` bullets are a conjunction smell: keep one event, or split the Scenario if it tests two behaviours.
   - `THEN` → `Then` (+ `And` per extra outcome).
   - Collapse Scenarios that differ only in data into a single `Scenario Outline:` with an `Examples:` table; lift a `Given` shared by all of the card's Scenarios into a `Background:`.
   - Third-person business language in the **spec's own language**; emit `# language: <code>` as the block's first line when not English (`# language: hu` for Hungarian specs).
   - No UI, route, or field-level detail. Declarative, not procedural. Steps sourced from the spec's `WHEN`/`THEN`, never from the card's prose.

5. **Add traceability.** Comment the block with its origin (`# Spec: <capability> › <Requirement name>`; append the change id when sourced from a delta) and tag it with `@EPIC-<epic-slug>` and `@STORY-…` taken from the map row (epic map, or an epic-less card's self-carried row) — verbatim, never re-minted (see `CONVENTIONS.md`; this skill CONSUMES the IDs). The Gherkin tags are the card's only in-body trace, so they must match the map exactly.

6. **Fill the card in place.** Replace the `BDD Test` placeholder in each story card with the fenced `gherkin` block. Diff before overwriting — never clobber hand-edited steps, and change **only** the `BDD Test` section, leaving every other section of the card untouched. Write no separate files. Completion: every targeted card's `BDD Test` section holds its Gherkin.

7. **Verify exhaustively.** Every source card has a filled `BDD Test` section; every `#### Scenario` its map row assigned maps to a Gherkin `Scenario`/`Scenario Outline` in that card; every generated Scenario passes the [`BDD-RULES.md`](BDD-RULES.md) checklist; every `@EPIC-…`/`@STORY-…` tag matches its map row (epic map / epic-less self-row) verbatim; nothing appears that the spec does not state. Report any bullet you could not map cleanly (e.g. a `THEN` whose precondition the spec never gives) rather than guessing.

## Next step

This is the last step of the planning chain — the backlog is now complete on disk. Offer (don't auto-run) `pte-openspec-jira-sync` to mirror the epics and cards into a Jira project through the Atlassian MCP.
