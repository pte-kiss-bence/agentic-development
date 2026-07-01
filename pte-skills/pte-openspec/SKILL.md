---
name: pte-openspec
description: Router for the pte-openspec-* pipeline — which skill turns OpenSpec specs into epics, story cards, and BDD tests, and in what order.
disable-model-invocation: true
---

The `pte-openspec-*` skills form one pipeline from OpenSpec specs to a groomed, testable backlog. Shared conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md).

## Pipeline

The stages run **strictly in order** — each consumes the artifact the previous one produced. The **epic** is the largest unit: it contains its stories in its description. The **story card** is the smallest unit: it contains its own BDD test(s) in its description. Nothing is written outside these two artifact kinds — there is no separate `features/` tree.

1. **Explore / propose** — `openspec-explore`, `openspec-propose` (or the `opsx:*` commands) turn an idea into a spec / change delta under `openspec/`.
2. **`pte-openspec-to-epics`** → `epics/` — rolls Requirements up into lean Agile epics; each epic lists its stories in its *User story-k* section. **MINTS** the `EPIC-…`/`STORY-…` trace IDs and the *Forrás-spec hivatkozás* map.
3. **`pte-openspec-to-stories`** → `stories/` — expands each epic story into a refinement-ready Agile story card, one per `STORY-…`. **CONSUMES** the epic's IDs. Leaves a **`BDD teszt`** section as a placeholder for stage 4.
4. **`pte-openspec-bdd-tests`** — fills each story card's **`BDD teszt`** section with declarative Gherkin, sourced from the spec scenarios the card's map row assigns. **CONSUMES** the story cards; edits them in place, writes no separate files.

Every stage after 2 keys off the same trace IDs, so an epic, its story cards, and the Gherkin embedded in each card line up one-to-one. The chain is linear: 4 depends on 3, which depends on 2.
