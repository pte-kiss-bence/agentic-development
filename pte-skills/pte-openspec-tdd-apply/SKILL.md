---
name: pte-openspec-tdd-apply
description: Build stage — implement an OpenSpec change test-first, each task through a red-green TDD loop. Invoke by name when the change is ready to build.
disable-model-invocation: true
---

Apply an OpenSpec change the test-first way. This skill owns nothing but the **ordering** between two existing skills: `openspec-apply-change` supplies the change selection, `contextFiles`, and the per-task loop; `tdd` supplies the red-green discipline that turns one task into tested code. Do not restate either here — invoke them by name.

The single rule this skill exists to enforce: a task's implementation never lands before its test goes **red**. Per task you run `tdd`'s vertical loop — one failing test, minimal code to pass, repeat per behaviour — never the horizontal shortcut of writing every test up front. Batch-all-tests is the anti-pattern `tdd` names: tests written in bulk verify _imagined_ behaviour and go insensitive to real breakage.

## Steps

1. **Enter the apply flow.** Run `openspec-apply-change` to select the change, read every file under `contextFiles`, and get the task list. _Done when:_ change selected, context files read, pending tasks known.

2. **Pick the next pending task.** Take one unchecked `- [ ]` task; read it and the spec behaviour it maps to. _Done when:_ exactly one task chosen and the behaviours it requires are listed.

3. **Run the red-green loop for that task.** Invoke `tdd`. For each behaviour the task requires: write ONE test that fails (**red**), then the minimal code that makes it pass (**green**); repeat. The first test is the **tracer bullet** — end-to-end through the public interface. Forbidden: writing all of the task's tests before any implementation. _Done when (checkable + exhaustive):_ every behaviour the task names has a test and all are green — none left red.

4. **Mark the task done.** Only now flip `- [ ]` → `- [x]` in the tasks file. _Done when:_ checkbox flipped for the task whose tests are all green.

5. **Loop or finish.** Return to step 2 while pending tasks remain. When none remain, hand back to `openspec-apply-change`'s completion output (progress, suggest archive).

Refactor only while green (see `tdd`) — never with a red test outstanding.

## Stop and ask — don't guess — when

- a task is ambiguous or its behaviours aren't clear from the spec;
- a test stays red and you can't reach green without a design change — suggest updating the change's artifacts;
- no test framework or runner exists, or you can't determine how to run the tests — do not fabricate one;
- implementation reveals the change's design is wrong.
