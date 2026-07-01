---
name: pte-openspec
description: Router for the pte-openspec-* pipeline — which skill turns OpenSpec specs into epics, story cards, and BDD tests, and in what order.
disable-model-invocation: true
---

The `pte-openspec-*` skills form one pipeline from OpenSpec specs to a groomed, testable backlog. Shared conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md).

## Pipeline

1. **Explore / propose** — `openspec-explore`, `openspec-propose` (or the `opsx:*` commands) turn an idea into a spec / change delta under `openspec/`.
2. **`pte-openspec-to-epics`** → `epics/` — rolls Requirements up into lean Agile epics. **MINTS** the `EPIC-…`/`STORY-…` trace IDs and the *Forrás-spec hivatkozás* map.
3. **`pte-openspec-to-stories`** → `stories/` — expands each epic story into a refinement-ready Agile story card. **CONSUMES** the epic's IDs.
4. **`pte-openspec-bdd-tests`** → `features/` — turns the spec's scenarios into declarative Gherkin `.feature` files, tagged with the same `@EPIC-…`/`@STORY-…` IDs. **CONSUMES** the epic's map.

Every stage after 2 keys off the same trace IDs, so an epic, its story cards, and its `.feature` files line up one-to-one. Stages 3 and 4 are independent consumers of stage 2 — run either or both.
