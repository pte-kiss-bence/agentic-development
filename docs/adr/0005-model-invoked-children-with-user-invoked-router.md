# pte-openspec: model-invoked child skills; user-invoked router and build stage

The `pte-openspec-*` family is split into a **router** (`pte-openspec`) that maps the pipeline, and the **child skills** that do the work (`-to-epics`, `-to-stories`, `-bdd-tests`, `-jira-sync`, `-dependency-graph`, `-tdd-execute`). The `writing-great-skills` reference frames a router as the cure for piled-up **cognitive load**: make the children *user-invoked* (they strip their descriptions, costing zero **context load**), and let one user-invoked router be the human's index into them. Following that pattern literally, the children would carry no descriptions and the agent could reach none of them on its own.

We deliberately do **not** do that for the planning/publishing children. They stay **model-invoked** (each keeps a description with trigger phrasing); **two** skills are user-invoked (`disable-model-invocation: true`): the **router** and the **build stage** (`pte-openspec-tdd-execute`).

## Why the planning children are model-invoked

The planning side is **agent-orchestrated, not human-orchestrated.** `CLAUDE.md` pins a post-propose flow: once `openspec-propose` finishes, the agent itself offers the right-sized next step and then fires the appropriate planning child (`to-epics` in mint/reconcile, `to-stories` epic-less, then `bdd-tests`, optionally `jira-sync` / `dependency-graph`). For that to work the agent must be able to **reach those children autonomously** — which is exactly what a model-facing description buys and what `disable-model-invocation` would remove. Making them user-invoked would force the human to type each skill name in sequence, defeating the orchestration `CLAUDE.md` exists to provide.

## Why the router and the build stage are user-invoked

- **`pte-openspec` (router)** — read by a person getting their bearings; it is a map, not a step the agent runs. Nothing autonomous needs to reach it (the orchestration lives in `CLAUDE.md`, which is always in context), so it pays no context load.
- **`pte-openspec-tdd-execute` (build stage)** — building is a phase a **human deliberately starts**, and the gate is a **hard, mechanical flag, not prose**: `disable-model-invocation` makes the Skill tool refuse the invocation even immediately after the user opted into the build path in the post-propose AskUserQuestion. That consequence is accepted on purpose — when a flow reaches the build step, the agent **instructs** ("type `/pte-openspec-tdd-execute`") instead of invoking, and the one manual keystroke is the point: no wording ambiguity, no model judgement call, can decide to start a build. `CLAUDE.md`'s post-propose paths and the router's step 4 say this explicitly so the offer never reads as something the agent will launch.

History: this ADR's original text was self-contradictory — it listed the build stage among the children the agent fires post-propose while also declaring it user-invoked. The flag was then briefly removed (prose-only gate) so the agent could fire the build after an opt-in; **reverted**: manual start is the desired property, and the docs (`CLAUDE.md`, router, README) now consistently describe the build step as typed by the human.

## The cost we accept

Every planning child's description sits in the context window every turn — the context-load cost the router pattern is designed to avoid. We accept it: the descriptions are pruned to one trigger per branch (per `writing-great-skills`), and the autonomy they enable is worth more than the tokens they cost. The router is therefore **not** a load-reduction device here; it is human orientation and the on-disk companion to the `CLAUDE.md` orchestration rules. On top of that, every build start costs one manual skill invocation — deliberate, see above. If the family grows enough that the aggregate description load becomes a real problem, revisit this — the fallback is to make the least-autonomously-triggered children user-invoked and lean harder on the router.
