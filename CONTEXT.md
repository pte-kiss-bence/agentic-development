# Agentic Development — pte-openspec pipeline

Ubiquitous language for the `pte-openspec-*` skill family: the pipeline that turns OpenSpec specs into a groomed, testable Agile backlog and then builds the change test-first. Terms are Hungarian where the generated artifacts are Hungarian; keep them verbatim.

## Language

**Epic**:
The largest unit of the backlog. A value-oriented strategic container that carries its stories in its own description (*User story-k* section). ~3–10 stories; more than ~10 is a signal it should be two or more epics.
_Avoid_: feature bucket, task list

**Story kártya** (story card):
The smallest unit of the backlog and the home of its own BDD test(s). One markdown file per `STORY-…`, carrying the `BDD teszt` section that holds its embedded Gherkin.
_Avoid_: ticket, task, issue

**BDD teszt** (embedded Gherkin):
The declarative Gherkin inside a story card's `BDD teszt` section. **Living documentation** now — human-readable behaviour, not executable in this pipeline — implemented later as a Playwright E2E test.
_Avoid_: executable test, .feature file, acceptance test (in the runnable sense)

**Planning chain**:
Stages 1–3 (`pte-openspec-to-epics` → `-to-stories` → `-bdd-tests`). Produces the human-facing backlog on shared trace IDs. Distinct from the build stage; the two do not feed each other.
_Avoid_: pipeline (the whole thing is the pipeline; this is one phase of it)

**Build stage**:
Stage 4 (`pte-openspec-tdd-apply`). Implements the OpenSpec change's **tasks** test-first via red-green TDD. Reads the tasks, not the planning artifacts, and does not key on trace IDs.
_Avoid_: implementation phase, apply step

**Trace ID**:
`EPIC-<epic-slug>` and `STORY-<epic-slug>-<requirement-slug>` (with a stable `-<aspect-slug>` when several stories split one requirement). Anchored on the requirement, never a running index.
_Avoid_: ticket number, index

**Forrás-spec hivatkozás** (traceability map):
The per-epic table (`Story ID | Epic ID | Forrás ### Requirement | Lefedett #### Scenario-k`) that records the requirement→story split. The cross-skill contract downstream skills read instead of re-deriving.
_Avoid_: mapping table, index

**MINTS / CONSUMES**:
Ownership of trace IDs. `pte-openspec-to-epics` **MINTS** (creates) the IDs; `-to-stories` and `-bdd-tests` **CONSUME** them (reference verbatim, never re-mint).
_Avoid_: generate/reuse (used loosely elsewhere)

**Spec** (source of truth):
The OpenSpec `### Requirement` / `#### Scenario` (`WHEN`/`THEN`) content. The single anchor both phases trace back to — behavioural text always comes from here, never from epic or card prose.
_Avoid_: requirements doc
