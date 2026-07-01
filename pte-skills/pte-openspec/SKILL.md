---
name: pte-openspec
description: Router for the pte-openspec-* pipeline — which skill turns OpenSpec specs into epics, story cards, and BDD tests, publishes the backlog to Jira, then implements the change test-first, and in what order.
disable-model-invocation: true
---

The `pte-openspec-*` skills form one pipeline from OpenSpec specs to a groomed, testable backlog — then on to a test-first implementation of the change. Shared conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md).

## Pipeline

The pipeline has **two phases** that share one anchor — the OpenSpec spec — but do **not** feed each other. The **planning chain** (steps 1–3) runs strictly in order, each consuming the artifact the previous one produced. The **build stage** (step 4) implements the change from its OpenSpec tasks, independently of the planning artifacts. The **epic** is the largest unit: it contains its stories in its description. The **story card** is the smallest unit: it contains its own BDD test(s) in its description. Nothing is written outside these two artifact kinds — there is no separate `features/` tree.

**Prerequisite — Explore / propose.** `openspec-explore`, `openspec-propose` (or the `opsx:*` commands) turn an idea into a spec / change delta under `openspec/`. Not a `pte-openspec-*` skill; it is the source the whole pipeline reads.

Planning chain (model-invoked; keys off the shared trace IDs):

1. **`pte-openspec-to-epics`** → `epics/` — rolls Requirements up into lean Agile epics; each epic lists its stories in its *User story-k* section. **MINTS** the `EPIC-…`/`STORY-…` trace IDs and the *Forrás-spec hivatkozás* map.
2. **`pte-openspec-to-stories`** → `stories/` — expands each epic story into a refinement-ready Agile story card, one per `STORY-…`. **CONSUMES** the epic's IDs. Leaves a **`BDD teszt`** section as a placeholder for step 3.
3. **`pte-openspec-bdd-tests`** — fills each story card's **`BDD teszt`** section with declarative Gherkin, sourced from the spec scenarios the card's map row assigns. **CONSUMES** the story cards; edits them in place, writes no separate files. The Gherkin is **living documentation**, implemented later as a Playwright E2E test.
   - ⇢ **(planned) Playwright E2E** — a future, separate phase implements the embedded Gherkin as Playwright E2E tests. No skill for it yet.

Publishing (optional; runs after the planning chain, before or alongside build):

- **`pte-openspec-jira-sync`** — mirrors `epics/` and `stories/` into a Jira project through the Atlassian MCP. **CONSUMES** the artifacts and their trace IDs (as `trace:…` labels); mints nothing. Runs on a **field-ownership** model — local owns content, Jira owns workflow state — so re-runs are idempotent and never clobber either side. Writes back a `## Jira szinkron` block into each card. Requires the Atlassian MCP to be wired and authenticated.

Build stage (user-invoked; independent of the planning artifacts):

4. **`pte-openspec-tdd-apply`** — implements the change's tasks test-first: per task a red-green loop (via `tdd`) before the checkbox flips. **CONSUMES** the OpenSpec change's `tasks` (through `openspec-apply-change`), **not** the trace IDs and **not** the epics/stories/Gherkin. **User-invoked** — it does not fire autonomously; run it by name once the change is ready to build.

The planning chain's steps key off the same trace IDs, so an epic, its story cards, and the Gherkin embedded in each card line up one-to-one. The build stage turns the change's tasks into tested code and does not touch the planning artifacts — the two phases meet only at the spec.
