Use the `/writing-great-skills` skill as your authoring reference to build out the `pte-openspec-*` skill pipeline in this repo: create the new `pte-openspec-to-stories` skill described below, and perform the pipeline unification (shared conventions reference + router + rewiring the two existing sibling skills onto the shared reference). Stay within that scope — do not add unrelated files or change repo config beyond what is specified. The sibling rewiring must be **behavior-preserving** (pointer swaps only, the rules' meaning unchanged). Diff before overwriting any file and never clobber unrelated hand-edited content.

## Skill to build

Name: `pte-openspec-to-stories`. Location: `.claude/skills/pte-openspec-to-stories/` (`SKILL.md` + a worked `EXAMPLE.md`). Sibling to `pte-openspec-to-epics` and `pte-openspec-bdd-tests` — read all of `pte-openspec-to-epics/SKILL.md` (and its `EXAMPLE.md`) and `pte-openspec-bdd-tests/SKILL.md` before writing, and match their voice, rigor, and structure exactly.

## Role in the pipeline — align tightly with the epics skill

The two skills are one pipeline: **`pte-openspec-to-epics` owns the epics and the skeletal stories inside them; this skill owns the detailed story card for each of those stories.** `pte-openspec-to-epics` already emits, per epic, a *User story-k* section (5–15 stories, each with a `STORY-…` ID + `Mint … szeretnék … hogy …` line + high-level acceptance criteria) and a *Forrás-spec hivatkozás* map table. This skill's job is to **expand each of those epic-defined stories into a full, refinement-ready Agile story card** — it does NOT re-invent the story set.

So the **primary input is the epic file(s)** produced by `pte-openspec-to-epics` (default `epics/`), not the raw spec. The raw spec is the **traced-through source** for the detailed acceptance criteria: for each story, follow the epic's *Forrás-spec hivatkozás* map to the exact `### Requirement` / `#### Scenario` blocks and expand their `WHEN`/`THEN` into full Given/When/Then.

## Design constraints (the generated skill MUST encode these)

1. **Invocation:** model-invoked (no `disable-model-invocation`). Write a description that front-loads the leading word and lists only genuinely distinct trigger branches (e.g. "wants detailed story cards for an epic's stories", "expand `pte-openspec-to-epics` stories into refinement-ready cards", "turn OpenSpec stories into a groomed backlog"). One trigger per branch — no synonym duplication. Make the description name the epic→story relationship so the agent reaches for it after the epics skill, not instead of it.

2. **Output language:** every story file's **content is Hungarian**, matching `openspec/config.yaml` and the sibling skills. Keep **verbatim, never translate:** code, identifiers, API names, CLI commands, file paths, `### Requirement` names, `#### Scenario` titles, `EPIC-…`/`STORY-…` trace IDs/slugs, and commit-type keywords (`feat`/`fix`/...). Write the story-card section headings in Hungarian.

3. **Best-practice fidelity — this is the skill's core value.** Base it on canonical Agile user-story practice (no authoritative Context7 library exists for this — do NOT fabricate a citation or cite a library). Each story card MUST carry:
   - **Trace header:** the story's `STORY-…` ID and its parent `EPIC-…` ID — both taken **verbatim** from the epic, establishing the up-link to the epic.
   - **Connextra statement:** `Mint [szerep], szeretnék [cél], hogy [érték]` — **reuse the epic's existing story line verbatim**; only sharpen wording if the epic's is a placeholder, and note when you do.
   - **Acceptance criteria** in Given/When/Then form (`Amennyiben` / `Amikor` / `Akkor`), expanded **1:1** from the `#### Scenario` `WHEN`/`THEN` bullets that the epic's *Forrás-spec hivatkozás* map assigns to this story — never invented, never pulled from a Scenario the map didn't assign to this story.
   - **INVEST self-check** — a short line confirming Independent, Negotiable, Valuable, Estimable, Small, Testable; flag any letter it fails.
   - **3 C's** framing — the card is a placeholder for conversation; acceptance criteria are the confirmation.
   - **Vertical slice** — restate the value slice this story delivers (carried from the epic), never a technical layer.
   - **Definition of Ready / Definition of Done** placeholders, **priority**, and an **estimate** placeholder (story points / relative size — left blank for team refinement, never guessed).
   - **Dependencies & risks** — including sibling stories within the same epic.

4. **Trace-ID contract — consume, never mint (the epics skill OWNS the IDs).** `STORY-<epic-slug>-<requirement-slug>` and `EPIC-<epic-slug>` are owned by `pte-openspec-to-epics`. This skill:
   - Reads the epic's *Forrás-spec hivatkozás* map and **reuses** each `STORY-…` ID and its `EPIC-…` parent verbatim, matching rows by verbatim `### Requirement` / `#### Scenario` title (mirror how `pte-openspec-bdd-tests` consumes that same map).
   - Produces **exactly one card per `STORY-…` row** in the epic map — no more, no fewer.
   - Every card cross-references, verbatim, its parent `EPIC-…`, its source `### Requirement`, and its covered `#### Scenario`s.
   - **Fallback only when no epic exists:** if the user points at a spec with no epic produced yet, the skill should say so and offer to run `pte-openspec-to-epics` first (preferred), or, if the user insists, derive its own requirement-anchored `STORY-<capability>-<requirement-slug>` IDs — clearly flagged as provisional until an epic reconciles them. This is a degraded path, not a co-equal mode.

5. **Traceability output:** a per-run table, one row per card — `STORY-…` ID, parent `EPIC-…`, source `### Requirement`, covered `#### Scenario`s (titles verbatim). This is the contract that lets an epic, its cards, and the BDD `.feature` files all line up on the same IDs.

6. **Steps section** — a copyable checklist with **checkable, exhaustive completion criteria** at the same rigor as `pte-openspec-to-epics`: resolve the epic file(s) → enumerate every `STORY-…` row and its mapped Scenarios → author one card per story (full anatomy, AC expanded from the mapped Scenarios) → write files → **verify exhaustively that every `STORY-…` in every source epic has exactly one card, and every `#### Scenario` the epic map assigned is covered by its card's acceptance criteria**; report any story or Scenario you could not place cleanly rather than guessing.

7. **Input resolution / plumbing:** locate the epic files (default `epics/`, configurable). Use **AskUserQuestion** when it's ambiguous which epic(s) to expand. When tracing Scenarios back to the spec, resolve the same source the epic used — a change delta (`openspec list --json` → `openspec/changes/<id>/specs/**/spec.md`) or a main spec (`openspec/specs/<capability>/spec.md`); pass `--store <id>` when the user names a store.

8. **Output:** one story card per file, nested to keep the epic→story hierarchy visible — default `stories/<epic-slug>/<story-slug>.md` (configurable), filename slugged from the `STORY-…` ID or title. **Diff before overwriting — never clobber hand-edited content.**

9. **`EXAMPLE.md`:** one complete worked transformation — take a small epic (with its *User story-k* + *Forrás-spec hivatkozás* map) as input, and show one full story card out: the reused `STORY-…`/`EPIC-…` IDs, the Connextra line carried from the epic, the acceptance criteria expanded from the mapped Scenarios, and the traceability row. Use progressive disclosure: keep `SKILL.md` lean and push the long worked example into `EXAMPLE.md`.

## Pipeline unification — make the three `pte-openspec-*` skills one coherent family

The three skills (`pte-openspec-to-epics` → `pte-openspec-to-stories` → `pte-openspec-bdd-tests`) currently each restate the same shared conventions (Hungarian-content + verbatim-identifier rule, the `EPIC-…`/`STORY-…` trace-ID grammar, diff-don't-clobber, `openspec list --json` / `--store` plumbing). That is **duplication** — a change to a shared rule means editing three files. Unify it into a **single source of truth**:

1. **Shared conventions reference** — create `.claude/skills/pte-openspec-shared/CONVENTIONS.md` (a disclosed reference file, not a runnable skill) holding, once, the rules every pipeline skill obeys: the Hungarian-content + verbatim-identifier list, the `EPIC-<epic-slug>` / `STORY-<epic-slug>-<requirement-slug>[-<aspect-slug>]` grammar and its ownership (`pte-openspec-to-epics` mints; the others consume), the *Forrás-spec hivatkozás* map as the cross-skill contract, diff-don't-clobber, and the OpenSpec source-resolution plumbing. The **new** `pte-openspec-to-stories/SKILL.md` MUST reach this file via a context pointer instead of re-stating those rules inline.

2. **Router / index skill** — create `.claude/skills/pte-openspec/SKILL.md` (user-invoked, `disable-model-invocation: true`) that names the pipeline, its order, and each stage's handoff: explore/propose → **epics** (owns IDs) → **stories** (expands each story to a card) → **bdd** (tags `.feature`s with the same IDs). It points at `CONVENTIONS.md` for the shared rules. This is the human-facing index of the family.

3. **Rewire the two existing siblings** (`pte-openspec-to-epics`, `pte-openspec-bdd-tests`) to consume `CONVENTIONS.md`: replace their duplicated inline shared-rule prose with a context pointer to it, so the family has one source of truth. This is **behavior-preserving** — the rules' meaning must not change; only the location does. Keep each skill's own unique content (the epics skill's ID-minting and epic anatomy, the bdd skill's Gherkin specifics) inline; move only what is genuinely shared. Diff each edit; do not alter behavior.

## Apply `/writing-great-skills` principles throughout

Optimize for **predictability** (same process every run). Give every step a checkable, where-needed exhaustive **completion criterion**. Keep each rule in a **single source of truth** — shared conventions live in `CONVENTIONS.md`; the IDs and the story set live in the epic; this skill references them, never redefines them. Prune **no-ops** sentence by sentence. Push the worked example down the information hierarchy into `EXAMPLE.md` behind a context pointer.

## Done when

- `.claude/skills/pte-openspec-to-stories/SKILL.md` and `EXAMPLE.md` exist and read as a natural sibling of the two existing `pte-openspec-*` skills, with the epic→story pipeline role stated explicitly.
- `.claude/skills/pte-openspec-shared/CONVENTIONS.md` holds the shared rules once, and the new stories skill points at it rather than duplicating them.
- `.claude/skills/pte-openspec/SKILL.md` is a user-invoked router naming the four-stage pipeline and its ID handoffs.
- The skill takes the **epic as primary input**, produces exactly one card per epic `STORY-…` row, reuses `STORY-…`/`EPIC-…` IDs verbatim (mints none), and expands acceptance criteria only from the Scenarios the epic map assigned.
- The Hungarian-content/verbatim-identifier rule, the Connextra + Given/When/Then + INVEST anatomy, and the exhaustive coverage verification are all encoded (inline or via the shared reference).
- Both existing sibling skills (`pte-openspec-to-epics`, `pte-openspec-bdd-tests`) now point at `CONVENTIONS.md` for the shared rules instead of restating them, with their behavior unchanged and their unique content left inline.
- `openspec/config.yaml` and unrelated files were not modified.
