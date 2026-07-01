# pte-openspec: model-invoked child skills, user-invoked router

The `pte-openspec-*` family is split into a **router** (`pte-openspec`) that maps the pipeline, and the **child skills** that do the work (`-to-epics`, `-to-stories`, `-bdd-tests`, `-jira-sync`, `-tdd-apply`). The `writing-great-skills` reference frames a router as the cure for piled-up **cognitive load**: make the children *user-invoked* (they strip their descriptions, costing zero **context load**), and let one user-invoked router be the human's index into them. Following that pattern literally, the children would carry no descriptions and the agent could reach none of them on its own.

We deliberately do **not** do that. The children stay **model-invoked** (each keeps a description with trigger phrasing); only the router and the build stage are user-invoked (`disable-model-invocation: true`).

## Why

The pipeline is **agent-orchestrated, not human-orchestrated.** `CLAUDE.md` pins a post-propose flow: once `openspec-propose` finishes, the agent itself offers the right-sized next step and then fires the appropriate child skill (`to-epics` in mint/reconcile, `to-stories` epic-less, `tdd-apply`). For that to work the agent must be able to **reach the children autonomously** — which is exactly what a model-facing description buys and what `disable-model-invocation` would remove. Making the children user-invoked would force the human to type each skill name in sequence, defeating the orchestration `CLAUDE.md` exists to provide.

So the two user-invoked skills are the two that a **human** deliberately triggers:

- **`pte-openspec` (router)** — read by a person getting their bearings; it is a map, not a step the agent runs. Nothing autonomous needs to reach it (the orchestration lives in `CLAUDE.md`, which is always in context), so it pays no context load.
- **`pte-openspec-tdd-apply` (build stage)** — user-invoked on purpose: building is a phase a human decides to start, not one the agent should launch on its own (see the router and [ADR 0001](0001-pte-openspec-two-phase-pipeline.md)).

## The cost we accept

This means every child description sits in the context window every turn — the context-load cost the router pattern is designed to avoid. We accept it: the descriptions are pruned to one trigger per branch (per `writing-great-skills`), and the autonomy they enable is worth more than the tokens they cost. The router is therefore **not** a load-reduction device here; it is human orientation and the on-disk companion to the `CLAUDE.md` orchestration rules. If the family grows enough that the aggregate description load becomes a real problem, revisit this — the fallback is to make the least-autonomously-triggered children user-invoked and lean harder on the router.
</content>
