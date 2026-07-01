---
name: pte-openspec
description: Router for the pte-openspec-* pipeline — which skill turns OpenSpec specs into epics, story cards, and BDD tests, then implements the change test-first, and in what order.
disable-model-invocation: true
---

The `pte-openspec-*` skills form one pipeline from OpenSpec specs to a groomed, testable backlog — then on to a test-first implementation of the change. Shared conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md).

## Pipeline

The stages run **strictly in order** — each consumes the artifact the previous one produced. The **epic** is the largest unit: it contains its stories in its description. The **story card** is the smallest unit: it contains its own BDD test(s) in its description. Nothing is written outside these two artifact kinds — there is no separate `features/` tree.

1. **Explore / propose** — `openspec-explore`, `openspec-propose` (or the `opsx:*` commands) turn an idea into a spec / change delta under `openspec/`.
2. **`pte-openspec-to-epics`** → `epics/` — rolls Requirements up into lean Agile epics; each epic lists its stories in its *User story-k* section. **MINTS** the `EPIC-…`/`STORY-…` trace IDs and the *Forrás-spec hivatkozás* map.
3. **`pte-openspec-to-stories`** → `stories/` — expands each epic story into a refinement-ready Agile story card, one per `STORY-…`. **CONSUMES** the epic's IDs. Leaves a **`BDD teszt`** section as a placeholder for stage 4.
4. **`pte-openspec-bdd-tests`** — fills each story card's **`BDD teszt`** section with declarative Gherkin, sourced from the spec scenarios the card's map row assigns. **CONSUMES** the story cards; edits them in place, writes no separate files.
5. **`pte-openspec-tdd-apply`** — implements the change's tasks test-first: per task a red-green loop (via `tdd`) before the checkbox flips. **CONSUMES** the OpenSpec change's `tasks` (through `openspec-apply-change`), not the trace IDs. The **build stage**, downstream of planning. **User-invoked** — it does not fire autonomously; run it by name once the change is ready to build.

Stages 2–4 are the planning chain: each is model-invoked and keys off the same trace IDs, so an epic, its story cards, and the Gherkin embedded in each card line up one-to-one. Stage 5 is the build stage — user-invoked, turns the change's tasks into tested code, and does not touch the epics/stories artifacts.
