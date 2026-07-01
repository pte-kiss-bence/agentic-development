Author a new skill for THIS repo (`@pte/agentic-development`) using the `writing-great-skills` skill as your reference. The new skill implements OpenSpec change tasks test-first, driving each task through the existing `tdd` skill.

<context>
- Repo conventions: workflow skills live in `.claude/skills/` and use the `pte-openspec-*` prefix (see `pte-openspec-to-epics`, `pte-openspec-to-stories`, `pte-openspec-bdd-tests`). Match their SKILL.md shape and frontmatter style.
- Two skills already exist and are the source of truth — DO NOT duplicate their content, point to them:
  - `.claude/skills/openspec-apply-change/SKILL.md` — the OpenSpec apply flow (`openspec status --json`, `openspec instructions apply --json`, read `contextFiles`, implement per task, flip `- [ ]` → `- [x]`, pause on ambiguity/blocker).
  - `.claude/skills/tdd/SKILL.md` — TDD best practice. Its HARD rule: NO horizontal slicing (never write all tests then all code). Correct approach = vertical slices / tracer bullets: one test → minimal impl → repeat, red-green per behavior, never refactor while red, expected values from independent source of truth.
- Read `.agents/skills/writing-great-skills/SKILL.md` and its `GLOSSARY.md` first, then apply its principles below.
</context>

<task>
Create `.claude/skills/pte-openspec-tdd-apply/SKILL.md` (add disclosed reference files only if a branch genuinely needs them). The skill orchestrates: take an OpenSpec change, and for EACH task run a red-green TDD loop before marking it done.
</task>

<research>
- Use the Context7 MCP tools (`resolve-library-id` then `query-docs`) to check for authoritative TDD / red-green guidance. If an official source surfaces, fold in only what is not already in `tdd/SKILL.md` and cite it. If nothing authoritative exists, state that `tdd` remains canonical and add no invented citations.
</research>

<skill_requirements>
Behavior the skill must encode:
1. Resolve the OpenSpec change and read its `contextFiles` by delegating to the `openspec-apply-change` flow — reference that skill, do not restate its CLI steps verbatim.
2. Per task, invoke the `tdd` skill and run its vertical red-green loop: write ONE failing test for a behavior, minimal code to pass, repeat per behavior in the task. Explicitly FORBID horizontal slicing (all tests up front).
3. Only after every behavior's test is green does the task count as done — then flip the tasks checkbox `- [ ]` → `- [x]`.
4. Completion criterion per task must be checkable and exhaustive: "every task behavior has a passing test and the tests are green," not "tests written."
5. Pause/stop conditions: ambiguous task, design issue revealed, red test that can't be made green, or missing test framework/runner → stop and ask, don't guess or fabricate a runner.
</skill_requirements>

<writing_great_skills_principles>
- Single source of truth: reference `tdd` and `openspec-apply-change` by name; do not copy their bodies. Duplication is a defect here.
- Choose invocation deliberately and state which: model-invoked (rich trigger description, e.g. "Use when the user wants to implement OpenSpec tasks test-first / TDD / red-green from a change") vs user-invoked (`disable-model-invocation: true`). Recommend model-invoked so the OpenSpec workflow chain can reach it; justify in one line.
- Use `tracer bullet` and `red`/`green` as leading words — do not re-explain them, let `tdd` own the definition.
- Give each step a checkable completion criterion. Push any long reference behind a context pointer (progressive disclosure) so SKILL.md stays legible.
- Prune no-ops: drop any line the agent already obeys by default.
</writing_great_skills_principles>

<constraints>
- Only create/edit files under `.claude/skills/pte-openspec-tdd-apply/`. Do not modify `tdd`, `openspec-apply-change`, other skills, or `CLAUDE.md`.
- Keep the skill body in English to match existing `pte-openspec-*` skills (the Hungarian-docs rule applies to generated artifacts, not skill definitions).
- Do not add features beyond what is listed. Do not scaffold tests, a runner, or example projects.
</constraints>

<done_when>
- `.claude/skills/pte-openspec-tdd-apply/SKILL.md` exists with valid frontmatter (`name`, `description`) and a stated invocation choice.
- The skill enforces per-task vertical red-green and explicitly forbids horizontal slicing.
- It references `tdd` and `openspec-apply-change` instead of duplicating them.
- Every step has a checkable completion criterion and stop/pause conditions are listed.
- Report: the invocation choice made (+ one-line why), any Context7 source used or "none authoritative — tdd canonical," and the file path created.
</done_when>

Read `writing-great-skills`, `tdd`, and `openspec-apply-change` SKILL.md files before writing. Stop and ask before creating any file outside the target skill folder.
