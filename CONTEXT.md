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
The per-epic table (`Story ID | Epic ID | Forrás ### Requirement | Lefedett #### Scenario-k`) that records the requirement→story split. The cross-skill contract downstream skills read instead of re-deriving.
_Avoid_: mapping table, index

**MINTS / CONSUMES**:
Ownership of trace IDs. `pte-openspec-to-epics` **MINTS** `EPIC-…` and their child `STORY-…` (in reconcile mode it reuses the epic's `EPIC-…` verbatim and mints only the new `STORY-…`). `-to-stories` **CONSUMES** them, except in **epic-less mode** where it mints its own `STORY-<capability-slug>-…`. `-bdd-tests` only **CONSUMES** (reference verbatim, never re-mint).
_Avoid_: generate/reuse (used loosely elsewhere)

**Spec** (source of truth):
The OpenSpec `### Requirement` / `#### Scenario` (`WHEN`/`THEN`) content. The single anchor both phases trace back to — behavioural text always comes from here, never from epic or card prose.
_Avoid_: requirements doc
