---
name: pte-openspec-bdd-tests
description: Generate declarative BDD test scenarios (Gherkin .feature files) from OpenSpec specs. Use when the user wants BDD, Gherkin, or Cucumber tests from OpenSpec requirements or a change's delta spec, mentions "feature files from specs", or wants spec scenarios turned into executable living documentation.
---

An OpenSpec spec already carries behaviour: each `### Requirement` holds `#### Scenario` blocks of `WHEN`/`THEN` bullets. That is BDD in embryo. This skill turns it into **declarative** Gherkin — `.feature` files that read as living documentation and stay stable when the implementation changes, never leaking UI or procedural detail.

`declarative` is the governing word: every generated step describes _what_ the system does, not _how_ a user clicks it. The full rule set and anti-patterns live in [`BDD-RULES.md`](BDD-RULES.md) — load it before writing any step. A complete worked transformation is in [`EXAMPLE.md`](EXAMPLE.md).

## Source mapping

OpenSpec and Gherkin line up almost one-to-one. The single gap is `Given`: OpenSpec scenarios state the trigger and outcome but rarely the precondition, so you must supply it.

| OpenSpec | Gherkin |
|----------|---------|
| `openspec/specs/<capability>/spec.md` (or a change delta spec) | one `.feature` file |
| `### Requirement: <name>` (its SHALL text) | `Rule: <name>` (+ `Feature:` for the capability) |
| `#### Scenario: <name>` | `Scenario:` — or `Scenario Outline:` when only data varies |
| (precondition implied by the Requirement / WHEN) | `Given …` — **added**, not present in the source |
| `- **WHEN** …` | `When …` (one event; extra bullets become `And` only if genuinely separate events) |
| `- **THEN** …` | `Then …` (extra bullets become `And`) |

## Steps

Copy this checklist and tick each item as you go — the verify step is exhaustive, not a glance:

```
- [ ] 1. Source spec file set named
- [ ] 2. Every Requirement and Scenario enumerated
- [ ] 3. Feature / Rule / Scenario layout chosen
- [ ] 4. Declarative steps written against BDD-RULES.md
- [ ] 5. Traceability comments added (+ epic trace-ID tags when an epic map exists)
- [ ] 6. .feature files written (diffed, not clobbered)
- [ ] 7. Step-definition stubs (only if a framework is named)
- [ ] 8. Every Requirement and Scenario verified present; unmappable bullets reported
```

1. **Resolve the source specs.** A change's delta (`openspec list --json` → pick → `openspec/changes/<id>/specs/**/spec.md`) or a main spec (`openspec/specs/<capability>/spec.md`). If the input is vague, list the options with **AskUserQuestion**. If the user names a store, pass `--store <id>` on `openspec` commands, as the other `openspec-*` skills do. Completion: the exact file set to convert is named.

2. **Enumerate every Requirement and Scenario** in those files — name, SHALL text, and each `WHEN`/`THEN` bullet. Completion (exhaustive): every Requirement and every Scenario in the source is on the list; none invented, none dropped.

3. **Lay out the Gherkin.** One `.feature` per capability; one `Rule:` per Requirement; one `Scenario:` per OpenSpec Scenario. Collapse Scenarios that differ only in data into a single `Scenario Outline:` with an `Examples:` table.

4. **Write declarative steps** against [`BDD-RULES.md`](BDD-RULES.md):
   - Supply the missing `Given` — the state the `WHEN` assumes.
   - `WHEN` → a single `When`. Multiple `WHEN` bullets are a conjunction smell: keep one event, or split the Scenario if it tests two behaviours.
   - `THEN` → `Then` (+ `And` per extra outcome).
   - Third-person business language in the **spec's own language**; emit `# language: <code>` as the first line when not English (e.g. `# language: hu` for Hungarian specs).
   - No UI, route, or field-level detail. Declarative, not procedural.

5. **Add traceability.** Comment each `Feature`/`Rule` with its origin so spec and test stay navigable: `# Spec: <capability> › <Requirement name>` (append the change id when sourced from a delta).

   **Epic trace tags (optional, only when an epic exists).** If `pte-openspec-to-epics` has produced an epic for this capability, read its "Forrás-spec hivatkozás" map and tag the Gherkin from it: `@EPIC-<epic-slug>` on the `Feature`/`Rule`, `@STORY-<epic-slug>-<n>` on each `Scenario`, matching rows by **verbatim Scenario title**. Take only the IDs from the epic — every step's text stays sourced from the spec, never the epic's prose. A Scenario absent from the map gets no story tag; report it. With no epic present, skip this and the skill runs standalone.

6. **Write the files** to the project's BDD directory (Cucumber default `features/<capability>/<requirement-slug>.feature`, or the configured one). Diff before overwriting — never clobber hand-edited steps.

7. **Step definitions are optional.** Scaffold stubs only when the user names a test framework or one is clearly in use; otherwise stop at the `.feature` living documentation.

8. **Verify exhaustively.** Every source Requirement and Scenario maps to a Gherkin `Scenario`/`Scenario Outline`, every generated Scenario passes the [`BDD-RULES.md`](BDD-RULES.md) checklist, and nothing appears that the spec does not state. Report any bullet you could not map cleanly (e.g. a `THEN` whose precondition the spec never gives) rather than guessing.
