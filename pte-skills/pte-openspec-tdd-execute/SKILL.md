---
name: pte-openspec-tdd-execute
description: Build stage — implement an OpenSpec change test-first, each task through a red-green TDD loop, informed by the task's tracing story (BDD acceptance criteria) and epic (cross-cutting constraints) when the backlog carries them. Invoke by name when the change is ready to build.
disable-model-invocation: true
---

Apply an OpenSpec change the test-first way. This skill owns nothing but the **ordering** between two existing skills: `openspec-apply-change` supplies the change selection, `contextFiles`, and the per-task loop; `tdd` supplies the red-green discipline that turns one task into tested code. Do not restate either here — invoke them by name.

The single rule this skill exists to enforce: a task's implementation never lands before its test goes **red**. Per task you run `tdd`'s vertical loop — one failing test, minimal code to pass, repeat per behaviour — never the horizontal shortcut of writing every test up front. Batch-all-tests is the anti-pattern `tdd` names: tests written in bulk verify _imagined_ behaviour and go insensitive to real breakage.

**Implement to the whole trace, not the task line alone.** A change's `tasks.md` is the unit of work, but a task is the leaf of a chain — **epic → story → spec requirement → task**. The task says _what_ to build; the **story** (`## User Story` + `## Context` + its `## BDD Test` Gherkin) says _for whom and why_, and its BDD scenarios **are the acceptance criteria** that define "done" for the behaviour; the **epic** (`## E2E Scenario`, `## Cross-cutting Concerns`) supplies constraints every task must honour. Read the tracing story and epic when the backlog carries them, and let them shape the tests and the implementation. This context is **best-effort**: the planning chain may have been skipped for a small change, so when no backlog artifact traces to the task, fall back to task + spec and proceed — never block on a missing card. Backlog layout and the `EPIC-…`/`STORY-…` trace grammar are in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md).

## Steps

1. **Enter the apply flow and locate the trace.** Run `openspec-apply-change` to select the change, read every file under `contextFiles`, and get the task list. Then, **when the backlog exists**, locate the story card(s) and parent epic(s) that trace to this change: the change's delta requirements (`openspec/changes/<id>/specs/**/spec.md`) name the `### Requirement`s; grep `openspec/backlog/` for the matching `STORY-…`/`EPIC-…` (a story is `STORY-<epic-slug>-<requirement-slug>` keyed on the requirement name, and the epic's *Source Spec Reference* map is the requirement→story index). Read the ones you find. _Done when:_ change selected, context files + task list read, and the tracing story/epic read if present (noted absent otherwise).

2. **Pick the next pending task.** Take one unchecked `- [ ]` task; read it, the spec behaviour it maps to, and — when present — the **story that owns that behaviour**: its `## User Story` (the user value), `## Context` (the value slice), and `## BDD Test` (the Gherkin that _is_ the acceptance criteria for this behaviour), plus the parent epic's `## E2E Scenario` and `## Cross-cutting Concerns` for constraints the task must honour. _Done when:_ exactly one task chosen, the behaviours it requires are listed, and (if present) its story's acceptance criteria and the epic's cross-cutting constraints are noted.

3. **Run the red-green loop for that task.** Invoke `tdd`. For each behaviour the task requires: write ONE test that fails (**red**), then the minimal code that makes it pass (**green**); repeat. When the story carries `## BDD Test` Gherkin, **encode those scenarios as the acceptance tests** rather than inventing criteria; the epic's `## Cross-cutting Concerns` (auth, i18n, error handling, …) are constraints every task honours, not separate tasks. The first test is the **tracer bullet** — end-to-end through the public interface. Forbidden: writing all of the task's tests before any implementation. _Done when (checkable + exhaustive):_ every behaviour the task names has a test and all are green — none left red.

4. **Mark the task done.** Only now flip `- [ ]` → `- [x]` in the tasks file. _Done when:_ checkbox flipped for the task whose tests are all green.

5. **Loop or finish.** Return to step 2 while pending tasks remain. When none remain, hand back to `openspec-apply-change`'s completion output (progress, suggest archive).

Refactor only while green (see `tdd`) — never with a red test outstanding.

## Stop and ask — don't guess — when

- a task is ambiguous or its behaviours aren't clear from the spec;
- the task or spec contradicts the tracing story's acceptance criteria (`## BDD Test`) or the epic's `## Cross-cutting Concerns` — surface the conflict, don't silently pick one;
- a test stays red and you can't reach green without a design change — suggest updating the change's artifacts;
- no test framework or runner exists, or you can't determine how to run the tests — do not fabricate one;
- implementation reveals the change's design is wrong.
