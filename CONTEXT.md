# Agentic Development — pte-openspec pipeline

Ubiquitous language for the `pte-openspec-*` skill family: the pipeline that turns OpenSpec specs into a groomed, testable Agile backlog and then builds the change test-first. Terms are Hungarian where the generated artifacts are Hungarian; keep them verbatim.

## Language

**Epic**:
The largest unit of the backlog. A value-oriented strategic container with English `##` headings and Hungarian body (`Description`, `Persona`, `E2E Scenario`, `Problem / Solution`, `Cross-cutting Concerns`, `MVP and Out of Scope`, `Success metrics`, `Risks and Dependencies`, `High Level Acceptance Criteria`, `Estimation` — the last a rollup of its stories' estimates). It carries its stories (*User story-k* section) and the *Forrás-spec hivatkozás* map in a trailing `<!-- pipeline-only -->` block — read by the downstream skills, stripped from Jira on sync. ~3–10 stories; more than ~10 is a signal it should be two or more epics.
_Avoid_: feature bucket, task list

**Story kártya** (story card):
The smallest unit of the backlog and the home of its own BDD test(s). One markdown file per `STORY-…`, with a lean schema (`Description`, `Context`, `BDD Test`, `Estimation`, `Risks and Dependencies`) whose acceptance criteria **are** the `BDD Test` Gherkin. It carries no trace row of its own (epic-driven: the map lives in the epic; epic-less: self-carried in a `<!-- pipeline-only -->` block).
_Avoid_: ticket, task, issue

**BDD Test** (embedded Gherkin):
The declarative Gherkin inside a story card's `BDD Test` section. **Living documentation** now — human-readable behaviour, not executable in this pipeline — implemented later as a Playwright E2E test.
_Avoid_: executable test, .feature file, acceptance test (in the runnable sense)

**INVEST** (mandatory story rule):
Every story card **must** satisfy INVEST — Independent, Negotiable, Valuable, Estimable, Small, Testable. The lean schema drops the printed *INVEST-ellenőrzés* section but **not** the rule: conformance is a hard gate enforced silently by `pte-openspec-to-stories` (and honored at split time by `pte-openspec-to-epics`). A slice that cannot be made INVEST-conform is re-split, never shipped.
_Avoid_: "INVEST was dropped" (only the section was; the rule stands)

**Estimation** (`## Estimation` — reference base):
The story card's estimate and the epic's rollup of them. **AI-authored** and, within the pipeline, **authoritative** (local-owned) — the *reference base* the team relies on. Managers layer their own internal/client **multipliers** on it **downstream**; that multiplier step is **out of pipeline scope** (the skills emit the raw base only). The hours are **not guessed** — they come from a **weighted complexity rubric** (a parametric factor model, not reference-class analogy). The full math (the rubric + its weights/`k`, the PERT formula, the Eβ→SP table, the 6h-ideal-day anchor, the split-warning threshold) is defined once in `pte-openspec-shared/CONVENTIONS.md`.
_Avoid_: cold-guessed hours, reference-class analogy (we use a tunable weighted rubric), "the team estimates it in Jira" (the AI base is authoritative; Jira's Story-points field is pushed from it), multiplied/client number (never emitted by the skills)

**PERT** (three-point becslés):
A story's base estimate as three points in **ideal engineer-hours** — `O` (optimista), `M` (valószínű), `P` (pesszimista) — yielding `Eβ = (O + 4M + P)/6` (the reference-base number) and `σ = (P − O)/6` (uncertainty). Lives in the card's `## Estimation`.
_Avoid_: single-point estimate, calendar time (it is *ideal* focused hours, before multipliers/overhead)

**Story point (SP)** (derived Fibonacci bucket):
A modified-Fibonacci size (1, 2, 3, 5, 8, 13, 20) **derived deterministically from `Eβ`** by the fixed table in `CONVENTIONS.md` — never guessed independently, so it can't contradict the hours. **Local-owned**: pushed to Jira's structured *Story points* field (Local → Jira), a deliberate flip from the old "estimate is Jira-owned".
_Avoid_: an independent estimate, a Jira-owned field

**Split-warning** (⚠, advisory):
A visible `⚠` note on a story whose estimate trips `SP ≥ 13` **or** `σ/Eβ > 0.5` (too big / too uncertain), flagging it as a split candidate. **Flag-only** — the card still ships; INVEST stays the qualitative gate, the estimate only surfaces the objective signal.
_Avoid_: hard gate, auto-split (it warns, it does not block)

**Planning chain**:
Stages 1–3 (`pte-openspec-to-epics` → `-to-stories` → `-bdd-tests`). Produces the human-facing backlog on shared trace IDs. Distinct from the build stage; the two do not feed each other.
_Avoid_: pipeline (the whole thing is the pipeline; this is one phase of it)

**Right-sizing** (a lánc a change súlyához igazítva):
The planning chain is not always run whole — it is entered at the depth the change warrants. A full feature runs `to-epics` in **mint** mode; a change landing in an existing epic runs `to-epics` in **reconcile** mode; a small standalone change skips `to-epics` and enters at `to-stories` in **epic-less** mode; a tweak/bug skips planning entirely (straight to `tdd-apply`). All variants converge on the same `to-stories → bdd-tests → tdd-apply → jira-sync` tail.
_Avoid_: "always run the full chain", ceremony-for-small-changes

**Mint mode / Reconcile mode** (`pte-openspec-to-epics`):
Two modes of the epics skill. **Mint** = greenfield: a new capability with no epic yet → mint new epic(s) and their `STORY-…`. **Reconcile** = a change delta whose capability already has an epic → reuse that `EPIC-…` verbatim and fold the delta in (mint the new `STORY-…`, extend an existing story's coverage for a modified scenario), diff-don't-clobber. Reconcile is the common case for a change request, since a change modifies existing behaviour.
_Avoid_: rewrite the epic, regenerate from scratch (reconcile edits in place)

**Epic-less story / epic-less mode** (`pte-openspec-to-stories`):
A first-class (not degraded) mode of the stories skill: a small standalone change that warrants a story but no epic → author the card(s) straight from the delta, skipping `to-epics`. The card leaves its parent `EPIC-…` line empty. Here the stories skill **MINTS** its own capability-namespaced `STORY-<capability-slug>-…`.
_Avoid_: provisional story, orphan story, degraded fallback

**Standalone collector epic** (`pte-openspec-jira-sync`):
The single reserved Jira Epic (`trace:EPIC-standalone`, e.g. *Önálló változtatások*) that parents epic-less stories in Jira so they are never orphaned. Created once, on demand.
_Avoid_: misc bucket, catch-all epic (it is a Jira-parent-of-record only)

**Build stage**:
Stage 4 (`pte-openspec-tdd-apply`). Implements the OpenSpec change's **tasks** test-first via red-green TDD. Reads the tasks, not the planning artifacts, and does not key on trace IDs.
_Avoid_: implementation phase, apply step

**Trace ID**:
`EPIC-<epic-slug>` and `STORY-<epic-slug>-<requirement-slug>` (with a stable `-<aspect-slug>` when several stories split one requirement). An epic-less story is namespaced on the capability instead: `STORY-<capability-slug>-<requirement-slug>` — equally stable, not provisional. Anchored on the requirement/capability, never a running index.
_Avoid_: ticket number, index

**Forrás-spec hivatkozás** (traceability map):
The per-epic table (`Story ID | Epic ID | Forrás ### Requirement | Lefedett #### Scenario-k`) that records the requirement→story split. Lives in the epic's `<!-- pipeline-only -->` block (hidden from Jira); the cross-skill contract downstream skills read instead of re-deriving. Story cards carry no trace row — they key off this map (epic-less cards self-carry an equivalent row).
_Avoid_: mapping table, index

**MINTS / CONSUMES**:
Ownership of trace IDs. `pte-openspec-to-epics` **MINTS** `EPIC-…` and their child `STORY-…` (in reconcile mode it reuses the epic's `EPIC-…` verbatim and mints only the new `STORY-…`). `-to-stories` **CONSUMES** them, except in **epic-less mode** where it mints its own `STORY-<capability-slug>-…`. `-bdd-tests` only **CONSUMES** (reference verbatim, never re-mint).
_Avoid_: generate/reuse (used loosely elsewhere)

**Spec** (source of truth):
The OpenSpec `### Requirement` / `#### Scenario` (`WHEN`/`THEN`) content. The single anchor both phases trace back to — behavioural text always comes from here, never from epic or card prose.
_Avoid_: requirements doc
