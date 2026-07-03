# pte-openspec pipeline — CONTEXT

Ubiquitous language for the `pte-openspec-*` skill family: the pipeline that turns OpenSpec specs into a groomed, testable Agile backlog and then builds the change test-first. One term, one meaning, one home: the definition lives here; each entry points at the file that owns the term's **mechanics**. Heading strings (`Source Spec Reference`, `Jira Sync`, …), trace IDs, and Hungarian artifact strings (`Eβ`, `Becsült munkaóra`) stay verbatim — they are the exact strings the artifacts and skills match on.

## The anchor

**Spec** (source of truth):
An OpenSpec specification under `openspec/specs/**/spec.md`, or a change's delta under `openspec/changes/<id>/specs/**/spec.md`. Structured as `### Requirement` → `#### Scenario` → `WHEN`/`THEN` bullets. The single source of **behaviour** both phases trace back to — behavioural text always comes from here, never from epic or card prose. Produced by `openspec-explore`/`openspec-propose` (not a `pte-openspec-*` skill). Resolution rules: [`CONVENTIONS.md` → Source-spec resolution](pte-openspec-shared/CONVENTIONS.md#source-spec-resolution).
_Avoid_: requirements doc

## The two phases

**Planning chain**:
Steps 1–3 (`pte-openspec-to-epics` → `-to-stories` → `-bdd-tests`), run **strictly in order**, each consuming the artifact the previous produced. Turns a spec into the human-facing backlog on shared trace IDs. Map: [`pte-openspec/SKILL.md`](pte-openspec/SKILL.md).
_Avoid_: pipeline (the whole thing is the pipeline; this is one phase of it)

**Build stage**:
Step 4 (`pte-openspec-tdd-execute`). Implements the OpenSpec change's **tasks** test-first via red-green TDD. **Write-decoupled** from planning, but reads the planning artifacts as **read-only, best-effort context**: the tracing story's `## BDD Test` Gherkin is the behaviour's acceptance criteria, the parent epic's `## Cross-cutting Concerns` are constraints; a missing card never blocks — it falls back to task + spec. See [ADR 0001](../docs/adr/0001-pte-openspec-two-phase-pipeline.md). **User-invoked** (`disable-model-invocation`): the human types it to start the build; the agent instructs, never fires ([ADR 0005](../docs/adr/0005-model-invoked-children-with-user-invoked-router.md)).
_Avoid_: "does not read the planning artifacts" (old, superseded framing), implementation phase, apply step, agent-started build

**Write-decoupled** (the phase relationship):
Neither phase writes the other's artifacts: the build stage never edits epics/stories/Gherkin, the planning chain never touches code or tasks. Reading across the boundary is allowed one way (build reads planning); the spec stays the sole behavioural source.
_Avoid_: "the phases do not feed each other" (reading does flow planning → build)

**Back-edge**:
The one exception to the chain's forward-only flow: an epic's `## Estimation` sums child-story estimates that don't exist until step 2, so step 1 emits a placeholder that a **deferred `pte-openspec-to-epics` re-run** (the *estimation-rollup pass*) fills. Mechanics: [`ESTIMATION.md` → Epic rollup](pte-openspec-shared/ESTIMATION.md).

## The two artifact kinds

**Epic**:
The **largest** unit: a value-oriented strategic container. English `##` headings, Hungarian body; the main narrative section is **`## Epic`** (never `## Description`). Jira-visible sections: `Epic`, `Persona`, `E2E Scenario`, `Problem / Solution`, `Cross-cutting Concerns`, `MVP and Out of Scope`, `Success metrics`, `Risks and Dependencies`, `High Level Acceptance Criteria`; the trailing pipeline-only fence holds `Estimation` (rollup), `Dependency Edges`, `User Stories`, `Source Spec Reference`. ~3–10 stories; more is a split signal. Governing word: `lean`. Anatomy: [`pte-openspec-to-epics/SKILL.md`](pte-openspec-to-epics/SKILL.md).
_Avoid_: feature bucket, task list, `## Description`

**Story kártya** (story card):
The **smallest** unit and the home of its own BDD test(s); one markdown file per `STORY-…`. Jira-visible sections: **`## User Story`** (never `## Description`), `## Context`, `## BDD Test`, `## Risks and Dependencies`; the pipeline-only fence holds the `Parent epic:` line, `## Estimation`, `## Dependency Edges` (epic-less cards also self-carry a `## Source Reference` row). The acceptance criteria **are** the `BDD Test` Gherkin. Embodies the **3 C's** (Card, Conversation, Confirmation). Governing word: `refinement-ready`. Anatomy: [`pte-openspec-to-stories/SKILL.md`](pte-openspec-to-stories/SKILL.md). There is **no separate `features/` tree**.
_Avoid_: ticket, task, issue

**BDD Test** (embedded Gherkin):
The declarative Gherkin inside a story card's `## BDD Test` section. **Living documentation** — human-readable behaviour, not executable in this pipeline — implemented later as a Playwright E2E test ([ADR 0002](../docs/adr/0002-embedded-gherkin-is-living-doc-for-later-playwright-e2e.md)). Governing word: `declarative` — every step says *what* the system does, never *how* a user clicks it; rules in [`BDD-RULES.md`](pte-openspec-bdd-tests/BDD-RULES.md).
_Avoid_: executable test, .feature file, acceptance test (in the runnable sense)

## Trace and contract

**Trace ID**:
`EPIC-<epic-slug>` and `STORY-<epic-slug>-<requirement-slug>` (a stable `-<aspect-slug>` when several stories split one requirement; epic-less: `STORY-<capability-slug>-…`, equally stable). Anchored on the requirement/capability, never a running index. The trace ID is also the artifact's **filename**, so a `grep`/glob finds it by name. Grammar: [`CONVENTIONS.md`](pte-openspec-shared/CONVENTIONS.md#trace-id-grammar-and-ownership).
_Avoid_: ticket number, index

**MINTS / CONSUMES**:
Ownership of trace IDs. `pte-openspec-to-epics` **MINTS** `EPIC-…` and their child `STORY-…` (in reconcile mode it reuses the epic's `EPIC-…` verbatim and mints only the new `STORY-…`). `-to-stories` **CONSUMES**, except in epic-less mode where it mints its own capability-namespaced `STORY-…`. `-bdd-tests`, `-jira-sync`, `-dependency-graph` only **CONSUME** (reference verbatim, never re-mint).
_Avoid_: generate/reuse (used loosely elsewhere)

**Source Spec Reference map** (traceability map; formerly *Forrás-spec hivatkozás*):
The per-epic table (`Story ID | Epic ID | Source ### Requirement | Covered #### Scenario`s) recording the requirement→story split — a judgement call, not derivable from the spec, which is why downstream skills read the map instead of re-deriving. Lives in the epic's pipeline-only fence; story cards carry no trace row (epic-less cards self-carry an equivalent `## Source Reference` row). Contract: [`CONVENTIONS.md`](pte-openspec-shared/CONVENTIONS.md#the-source-spec-reference-map--cross-skill-contract).
_Avoid_: mapping table, index

**Pipeline-only block**:
The `<!-- pipeline-only:start -->`/`<!-- pipeline-only:end -->` fence at the end of every epic and card, wrapping content the pipeline needs but Jira must not see as Description text: the epic's `User Stories` + `Source Spec Reference`, a card's `Parent epic:` line (+ epic-less `Source Reference` row), and every artifact's `## Estimation` and `## Dependency Edges`. `pte-openspec-jira-sync` strips the whole region from the pushed Description; `## Estimation` is the one fenced section whose **numbers** still reach Jira — via the structured fields, not the Description. Mechanics: [`CONVENTIONS.md` → Pipeline-only blocks](pte-openspec-shared/CONVENTIONS.md#pipeline-only-hidden-from-jira-blocks).

**Dependency Edges block**:
The machine-readable `## Dependency Edges` section (pipeline-only) each epic and card owns, declaring its **outbound** edges. Four ASCII tags — `depends-on` / `blocks` / `relates-to` (backticked target trace ID) and `external` (free text, graph-only) — so edges parse without reading Hungarian prose. The **single source of truth for edges**: `pte-openspec-dependency-graph` renders them, `pte-openspec-jira-sync` mirrors them as Jira issue links; the Jira-visible `## Risks and Dependencies` prose is never parsed. `depends-on`/`blocks` are inverses (deduped to one edge); a dangling target is warned-and-skipped. Contract: [`CONVENTIONS.md`](pte-openspec-shared/CONVENTIONS.md#the-dependency-edges-block--the-edge-contract).

## The three planning modes

Chosen mechanically by *does an epic already cover this capability?* — see **right-sizing**.

**Mint mode** (`pte-openspec-to-epics`):
Greenfield — a new capability with no epic yet → mint new epic(s) and their `STORY-…` from scratch.
_Avoid_: regenerate from scratch onto an existing epic

**Reconcile mode** (`pte-openspec-to-epics`):
A change delta whose capability **already has an epic** → reuse that `EPIC-…` verbatim, fold the delta in (mint new `STORY-…` for added requirements, extend an existing story's coverage for modified scenarios), diff-don't-clobber. The common case for a change request.
_Avoid_: rewrite the epic

**Epic-less mode** (`pte-openspec-to-stories`):
A small standalone change that warrants a story but no epic → skip `to-epics`; the stories skill mints its own capability-namespaced `STORY-…` straight from the delta, `Parent epic:` line left empty. A **first-class mode, not a degraded fallback** ([ADR 0003](../docs/adr/0003-right-size-the-planning-chain-per-change.md)).
_Avoid_: provisional story, orphan story, degraded fallback

## Governing principles

**Right-sizing**:
Enter the chain at the depth the change warrants; never run it whole out of ceremony. Full feature → mint; change into an existing epic → reconcile; small standalone → epic-less; tweak/bug → skip planning, straight to the build stage. All planning variants converge on the same **`to-stories` → `bdd-tests` → `jira-sync`** tail; the build stage is **not part of the tail** — a separate phase that runs whenever the change is ready, independent of publishing. Rationale: [ADR 0003](../docs/adr/0003-right-size-the-planning-chain-per-change.md).
_Avoid_: "always run the full chain", a tail that includes the build stage

**Diff-don't-clobber**:
Diff before overwriting; never clobber hand-edited content. Load-bearing for reconcile mode and every in-place edit (`bdd-tests`, `jira-sync`, the rollup pass). [`CONVENTIONS.md` → Writing output files](pte-openspec-shared/CONVENTIONS.md#writing-output-files).

**INVEST** (mandatory story rule):
Every story card **must** satisfy INVEST (Independent, Negotiable, Valuable, Estimable, Small, Testable). A **rule, not a printed section**: the lean card carries no *INVEST-ellenőrzés* block, but conformance is a hard silent gate — a slice that cannot be made conform is re-cut, never shipped. [`CONVENTIONS.md` → INVEST](pte-openspec-shared/CONVENTIONS.md#invest--the-story-gate).
_Avoid_: "INVEST was dropped" (only the section was; the rule stands)

**Output language**:
Every generated **artifact's content is Hungarian**; skill prose (`SKILL.md`) is English. **All `##` headings are English** — including the pipeline-only ones and `## Jira Sync`. Code, identifiers, CLI, `### Requirement`/`#### Scenario` titles, trace IDs, and commit-type keywords stay verbatim. [`CONVENTIONS.md` → Output language](pte-openspec-shared/CONVENTIONS.md#output-language).

## Estimation

**Estimation / reference base** (`## Estimation`):
The story card's estimate and the epic's rollup of them. **AI-authored** and, within the pipeline, **authoritative** (local-owned) — the *reference base* managers layer their internal/client **multipliers** on **downstream**; that multiplier step is out of pipeline scope (the skills emit the raw base only). The hours are **not guessed**: they come from a **weighted complexity rubric** (parametric factor model). The full math — rubric, weights/`k`, PERT formula, Eβ→SP table, 6h-ideal-day anchor, split-warning threshold, rollup rule, rendered format — is the single source of truth in [`ESTIMATION.md`](pte-openspec-shared/ESTIMATION.md); the ownership rule (SP local-owned, pushed to Jira) in [`CONVENTIONS.md`](pte-openspec-shared/CONVENTIONS.md#estimation--the-reference-base-pert--story-points). Decision: [ADR 0004](../docs/adr/0004-story-estimation-pert-plus-story-points-reference-base.md).
_Avoid_: cold-guessed hours, "the team estimates it in Jira", multiplied/client number (never emitted)

**PERT** (three-point becslés):
`O`/`M`/`P` in **ideal engineer-hours** → `Eβ = (O + 4M + P)/6`, `σ = (P − O)/6`. `M` comes from the rubric; the σ-drivers set the O↔P spread. Anchor: 6 ideal engineer-hours = 1 ideal day.
_Avoid_: single-point estimate, calendar time

**Story point (SP)**:
A modified-Fibonacci size **derived deterministically from `Eβ`** by the fixed table in `ESTIMATION.md` — never guessed independently, so the two views can't contradict. **Local-owned**: pushed to Jira's structured *Story points* field.
_Avoid_: an independent estimate, a Jira-owned field

**Split-warning** (⚠, advisory):
A visible `⚠` note when `SP ≥ 13` **or** `σ/Eβ > 0.25` (too big / too uncertain — the σ branch trips at u > 0.75 on the σ-driver scale), flagging a split candidate. **Flag-only** — the card still ships; INVEST stays the qualitative gate. Threshold SSOT: [`ESTIMATION.md`](pte-openspec-shared/ESTIMATION.md).
_Avoid_: hard gate, auto-split

## BDD and build

**Red-green**:
The `tdd` loop the build stage runs per task: one failing test (**red**) → minimal code to pass (**green**) → repeat per behaviour. The forbidden **horizontal shortcut** is writing every test up front.

**Tracer bullet**:
The first test of a task: end-to-end through the public interface, proving the whole path before filling in behaviours.

## Publishing

**Field-ownership**:
The governing word for `pte-openspec-jira-sync`: sync is not a direction but a set of fields, each with **exactly one owner**. Local owns the authored content (Summary, Description, labels, parent, Story points, Original estimate, dependency links); Jira owns the workflow state (status, assignee, sprint, comments, key). Each field flows from its owner only, so re-runs are idempotent. Table: [`pte-openspec-jira-sync/SKILL.md`](pte-openspec-jira-sync/SKILL.md#field-ownership--the-model-the-skill-runs-on).

**Standalone collector epic**:
The single reserved Jira Epic (`trace:EPIC-standalone`, e.g. *Önálló változtatások*) that parents epic-less stories in Jira so they are never orphaned. Created once, on demand.
_Avoid_: misc bucket, catch-all epic (it is a Jira-parent-of-record only)

**Jira Sync block**:
The `## Jira Sync` section (English heading — formerly `## Jira szinkron`) that `pte-openspec-jira-sync` writes back into each artifact, carrying the Jira-owned fields (key, issue type, status, URL, last sync) pulled from the board.
