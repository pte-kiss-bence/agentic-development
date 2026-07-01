# Glossary — the pte-openspec pipeline's ubiquitous language

The shared vocabulary the `pte-openspec-*` skills, `CLAUDE.md`, and the ADRs all speak. One term, one meaning, one home. Each entry defines the term and points at the file that owns its **mechanics** — the definition lives here so the skills can name the concept without re-defining it, and a *leading word* (a term the model already carries) keeps the whole suite anchored on the same behaviour.

Hungarian terms (`Forrás-spec hivatkozás`, `User story-k`, `Jira szinkron`, `Eβ`) stay verbatim — they are the exact strings the artifacts and skills match on.

## The anchor

- **spec** — an OpenSpec specification under `openspec/specs/**/spec.md`, or a change's delta under `openspec/changes/<id>/specs/**/spec.md`. Structured as `### Requirement` → `#### Scenario` → `WHEN`/`THEN` bullets. It is the **single source of behaviour** the whole pipeline reads, and the one anchor the two phases share. Produced by `openspec-explore`/`openspec-propose` (not a `pte-openspec-*` skill). Resolution rules: [`CONVENTIONS.md` → Source-spec resolution](../pte-skills/pte-openspec-shared/CONVENTIONS.md#source-spec-resolution).

## The two phases

- **planning chain** — steps 1–3 (`pte-openspec-to-epics` → `-to-stories` → `-bdd-tests`), run **strictly in order**, each consuming the artifact the previous produced. Turns a spec into a groomed, testable backlog. Map: [`pte-openspec/SKILL.md`](../pte-skills/pte-openspec/SKILL.md).
- **build stage** — step 4 (`pte-openspec-tdd-apply`), a **separate phase** off the same spec that implements the change's tasks test-first. It does **not** consume the planning artifacts; the two phases meet only at the spec.
- **back-edge** — the one exception to the chain's forward-only flow: an epic's `## Estimation` sums child-story estimates that don't exist until step 2, so step 1 emits a placeholder that a **deferred `pte-openspec-to-epics` re-run** fills (the *estimation rollup*). See [`ESTIMATION.md` → Epic rollup](../pte-skills/pte-openspec-shared/ESTIMATION.md).

## The two artifact kinds

- **epic** — the **largest** unit: a value-oriented strategic container (English `##` headings, Hungarian body) that carries its stories in its own description. Governing word: `lean`. Anatomy: [`pte-openspec-to-epics/SKILL.md`](../pte-skills/pte-openspec-to-epics/SKILL.md).
- **story card** — the **smallest** unit: a `refinement-ready` Agile card that carries its own BDD test(s) in its description. Embodies the **3 C's** (Card, Conversation, Confirmation — the `## BDD Test` is the Confirmation). Anatomy: [`pte-openspec-to-stories/SKILL.md`](../pte-skills/pte-openspec-to-stories/SKILL.md). There is **no separate `features/` tree** — the BDD test is embedded in the card.

## Trace and contract

- **trace ID** — the identifier that keys the planning chain together so an epic, its cards, and their Gherkin line up 1:1. `EPIC-<epic-slug>` and `STORY-<epic-slug>-<requirement-slug>` (epic-less: `STORY-<capability-slug>-…`). Grammar: [`CONVENTIONS.md` → Trace-ID grammar](../pte-skills/pte-openspec-shared/CONVENTIONS.md#trace-id-grammar-and-ownership). The trace ID is also the **filename** of both artifact kinds, so a `grep`/glob finds an epic or card by name.
- **MINTS / CONSUMES** — the two ownership verbs for trace IDs. A skill that **MINTS** creates the ID (`pte-openspec-to-epics`; `pte-openspec-to-stories` only in epic-less mode); one that **CONSUMES** reuses it verbatim and never re-mints (everything else). Ownership table: [`CONVENTIONS.md`](../pte-skills/pte-openspec-shared/CONVENTIONS.md#trace-id-grammar-and-ownership).
- **Forrás-spec hivatkozás map** — the traceability table (`Story ID | Epic ID | Forrás ### Requirement | Lefedett #### Scenario-k`) that `pte-openspec-to-epics` emits per epic. The **cross-skill contract**: downstream skills read the requirement→story→scenario split from this map instead of re-deriving it (the split is a judgement call, not derivable from the spec alone). Cards carry no trace row of their own; the map holds it. Contract: [`CONVENTIONS.md`](../pte-skills/pte-openspec-shared/CONVENTIONS.md#the-forrás-spec-hivatkozás-map--cross-skill-contract).
- **pipeline-only block** — a `<!-- pipeline-only:start -->`/`<!-- pipeline-only:end -->` fence at the end of a file wrapping content the pipeline needs but Jira must not see (the epic's *User story-k* + *Forrás-spec hivatkozás* map, or an epic-less card's self-carried *Forrás-hivatkozás* row). `pte-openspec-jira-sync` strips the whole region before push. Mechanics: [`CONVENTIONS.md` → Pipeline-only blocks](../pte-skills/pte-openspec-shared/CONVENTIONS.md#pipeline-only-hidden-from-jira-blocks).

## The three planning modes

Chosen mechanically by *does an epic already cover this capability?* — see **right-sizing**.

- **mint** — greenfield: a new capability with no epic yet → `pte-openspec-to-epics` mints new epic(s) from scratch.
- **reconcile** — a change delta whose capability **already has an epic** → `pte-openspec-to-epics` reuses that `EPIC-…` verbatim and folds the delta in (mint new `STORY-…` for added requirements, extend coverage for modified ones), diff-don't-clobber.
- **epic-less** — a small standalone change that warrants a story but no epic → skip `to-epics`; `pte-openspec-to-stories` mints its own capability-namespaced `STORY-…` straight from the delta, parent `EPIC-…` line left empty. A **first-class mode, not a degraded fallback** (see [ADR 0003](adr/0003-right-size-the-planning-chain-per-change.md)).

## Governing principles

- **right-sizing** — enter the chain at the depth the change warrants; never run it whole out of ceremony. Full feature → mint; change into an existing epic → reconcile; small standalone → epic-less; tweak/bug → skip planning, go to the build stage. Rationale: [ADR 0003](adr/0003-right-size-the-planning-chain-per-change.md).
- **diff-don't-clobber** — diff before overwriting; never overwrite hand-edited content. The load-bearing rule for reconcile mode and for every in-place edit (`bdd-tests`, `jira-sync`). [`CONVENTIONS.md` → Writing output files](../pte-skills/pte-openspec-shared/CONVENTIONS.md#writing-output-files).
- **INVEST** — the qualitative gate every story card must pass (Independent, Negotiable, Valuable, Estimable, Small, Testable). Enforced as a **rule, not a printed section**: a card that cannot be made INVEST-conform is a signal to re-cut the split, not to ship. [`CONVENTIONS.md` → INVEST](../pte-skills/pte-openspec-shared/CONVENTIONS.md#invest--the-story-gate).
- **output language** — every generated **artifact's content is Hungarian**; skill prose (`SKILL.md`) is English. Identifiers, code, CLI, `### Requirement`/`#### Scenario` titles, trace IDs, and commit-type keywords stay verbatim (never translated). [`CONVENTIONS.md` → Output language](../pte-skills/pte-openspec-shared/CONVENTIONS.md#output-language).

## Estimation

- **reference base** — the AI-authored, **local-owned** estimate the pipeline emits. Authoritative within the pipeline; the managers' internal/client **multipliers** ride on top **downstream** and are out of pipeline scope (the skills emit the raw base only). Model: [`ESTIMATION.md`](../pte-skills/pte-openspec-shared/ESTIMATION.md).
- **PERT three-point** — `O`/`M`/`P` (optimista/valószínű/pesszimista) ideal engineer-hours → `Eβ = (O + 4·M + P)/6` and `σ = (P − O)/6`. `M` comes from a **weighted complexity rubric**, not a cold guess. Anchor: 6 ideal engineer-hours = 1 ideal day.
- **Story Points (SP)** — derived **deterministically from Eβ** by a fixed table (never guessed independently). Local-owned, pushed to Jira's structured *Story points* field.
- **split-warning** — an advisory `⚠` (flag-only, not a gate) when `SP ≥ 13` or `σ/Eβ > 0.5`: too big or too uncertain, a split candidate. INVEST stays the actual gate.

## BDD and build

- **declarative** — the governing word for the embedded Gherkin: every step describes *what* the system does, never *how* the user clicks it. UI/procedural detail lives in the future Playwright E2E layer, never in the Gherkin. Rules: [`BDD-RULES.md`](../pte-skills/pte-openspec-bdd-tests/BDD-RULES.md).
- **living documentation** — the embedded Gherkin: human-readable behaviour, **not executable** in this pipeline. A later, separate phase implements it as a **Playwright E2E** test (no skill yet). See [ADR 0002](adr/0002-embedded-gherkin-is-living-doc-for-later-playwright-e2e.md).
- **red-green** — the `tdd` loop the build stage runs per task: one failing test (**red**) → minimal code to pass (**green**) → repeat per behaviour. The forbidden **horizontal shortcut** is writing every test up front.
- **tracer bullet** — the first test of a task: end-to-end through the public interface, proving the whole path before filling in behaviours.

## Publishing

- **field-ownership** — the governing word for `pte-openspec-jira-sync`: sync is not a direction but a set of fields, each with **exactly one owner**. Local owns the content it authored (Summary, Description, labels, parent, Story points); Jira owns the workflow state a team adds (status, assignee, sprint, comments, key). Each field flows from its owner only, so re-runs are idempotent. Table: [`pte-openspec-jira-sync/SKILL.md`](../pte-skills/pte-openspec-jira-sync/SKILL.md#field-ownership--the-model-the-skill-runs-on).
- **standalone collector epic** — the reserved Jira epic (`trace:EPIC-standalone`) that epic-less stories are parented under so they are never orphaned on the board.
- **Jira szinkron block** — the `## Jira szinkron` section `pte-openspec-jira-sync` writes back into each card, carrying the Jira-owned fields (key, status, URL) pulled from the board.
</content>
</invoke>
