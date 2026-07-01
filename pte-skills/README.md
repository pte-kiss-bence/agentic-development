# pte-skills — OpenSpec → groomed backlog pipeline

The `pte-openspec-*` skill family: a **two-phase** pipeline that turns [OpenSpec](https://github.com/fission-ai/openspec) specs into a groomed, testable Agile backlog (**planning chain**), then implements the change **test-first** (**build stage**). Not application code — Claude Code skills. The two phases share one anchor — the **spec** — but do **not** feed each other.

The skills' prose (`SKILL.md`) is English; every **generated artifact's content is Hungarian** (per `openspec/config.yaml`). Two artifact kinds only: the **epic** carries its stories in its description; the **story card** carries its own BDD test(s). There is **no separate `features/` tree**.

> This README is orientation only. The authoritative sources are the **router** ([`pte-openspec/SKILL.md`](pte-openspec/SKILL.md) — the map), the **shared rules** ([`pte-openspec-shared/CONVENTIONS.md`](pte-openspec-shared/CONVENTIONS.md)), and the **glossary** ([`../docs/glossary.md`](../docs/glossary.md) — the ubiquitous language). This file does not restate them.

## Workflow at a glance

```
                 openspec explore / propose            (prerequisite)
                            │
                            ▼
              openspec/specs/**/spec.md  ◀── shared anchor ──┐
              (### Requirement → #### Scenario → WHEN/THEN)   │
                            │                                 │
        ┌─── PLANNING ──────┘                                 │
        ▼                                                     │
   ┌──────────────────────────────┐                          │
   │  1. pte-openspec-to-epics     │  MINTS: EPIC-… / STORY-… │
   │            → epics/           │        + Forrás-spec map │
   └──────────────────────────────┘                          │
                  │  CONSUMES: epic map + IDs                 │
                  ▼                                           │
   ┌──────────────────────────────┐                          │
   │  2. pte-openspec-to-stories   │  one card / STORY-…      │
   │           → stories/          │  BDD Test = placeholder  │
   └──────────────────────────────┘                          │
                  │  CONSUMES: story cards                    │
                  ▼                                           │
   ┌──────────────────────────────┐                          │
   │  3. pte-openspec-bdd-tests    │  Gherkin into the card,  │
   │  (in place — no features/)    │  in place (no new file)  │
   └──────────────────────────────┘                          │
                  ╎ living doc                                │
                  ⇢ (planned) Playwright E2E — no skill yet   │
                  │  CONSUMES: epics/ + stories/ (trace labels)
                  ▼  (optional publish step)                  │
   ┌──────────────────────────────┐                          │
   │  ✳ pte-openspec-jira-sync     │  mirror to Jira          │
   │  (Atlassian MCP,              │  field-ownership,        │
   │   idempotent re-sync)         │  ## Jira szinkron block  │
   └──────────────────────────────┘                          │
                                                              │
        ┌─── BUILD (separate branch, independent) ────────────┘
        ▼   CONSUMES: change tasks (not the trace IDs, not epics/stories)
   ┌──────────────────────────────┐
   │  4. pte-openspec-tdd-execute    │  build stage: implements the
   │   (red-green tdd per task,    │  change's tasks test-first
   │    on openspec-apply-change)  │  (user-invoked)
   └──────────────────────────────┘
```

> In the diagram `epics/` and `stories/` are shorthand: the real output is **`openspec/backlog/epics/`** and **`openspec/backlog/stories/`** — co-located with the specs, inside the OpenSpec tree (but not OpenSpec-CLI-managed). See *Output location* in [`CONVENTIONS.md`](pte-openspec-shared/CONVENTIONS.md).

> **Optional read-only view** — `pte-openspec-dependency-graph` hangs off the backlog without extending the chain: it reads every card's `## Dependency Edges` block and renders a Mermaid dependency graph to `openspec/backlog/DEPENDENCY_GRAPH.md`. Run it any time after step 2; it authors nothing.

## Two phases, right-sized to the change

- **Planning chain** (steps 1–3) runs **strictly in order**, each step consuming the previous step's artifact. Its one **back-edge**: the epic's `## Estimation` sums child stories that only exist after step 2, so step 1 emits a placeholder that a deferred `pte-openspec-to-epics` re-run fills.
- **Build stage** (step 4) is a **separate phase** off the same spec — it implements the change's tasks, independently of the planning artifacts.

The chain is **not always run whole** — enter at the depth the change warrants (`mint` / `reconcile` / `epic-less` / skip-to-build). The mechanics of the modes and the right-sizing decision live in the **router** and in [ADR 0003](../docs/adr/0003-right-size-the-planning-chain-per-change.md); the term definitions in the [glossary](../docs/glossary.md).

## Folder structure

```
pte-skills/
├─ README.md                       ← this file (orientation)
├─ pte-openspec/SKILL.md           ← router (the map)
├─ pte-openspec-shared/
│  ├─ CONVENTIONS.md               ← shared rules (single source of truth)
│  └─ ESTIMATION.md                ← PERT + Story-Points model (rubric, formula, table)
├─ pte-openspec-to-epics/          ← SKILL.md + EXAMPLE.md
├─ pte-openspec-to-stories/        ← SKILL.md + EXAMPLE.md
├─ pte-openspec-bdd-tests/         ← SKILL.md + BDD-RULES.md + EXAMPLE.md
├─ pte-openspec-jira-sync/         ← SKILL.md + EXAMPLE.md
├─ pte-openspec-dependency-graph/  ← SKILL.md + EXAMPLE.md (optional read-only view)
└─ pte-openspec-tdd-execute/SKILL.md ← build stage (no EXAMPLE.md — emits code, not artifacts)
```

Generated artifacts live **outside this folder**, inside the OpenSpec tree beside the specs:

```
openspec/
├─ specs/            ← OpenSpec specs (the pipeline's source)
├─ changes/          ← OpenSpec changes (delta spec + tasks)
└─ backlog/          ← the generated backlog (not OpenSpec-CLI-managed)
   ├─ epics/         ← EPIC-<epic-slug>.md
   └─ stories/       ← <epic-slug>/STORY-<epic-slug>-<requirement-slug>.md (embedded BDD tests)
```

Each **artifact-producing** skill ships an `EXAMPLE.md` (full worked transformation); `pte-openspec-tdd-execute` emits code, so it has none. `pte-openspec-bdd-tests` adds `BDD-RULES.md` (the declarative-Gherkin rule set), loaded before it writes.
</content>
