---
name: pte-openspec-jira-sync
description: Sync the pipeline's epics and story cards to Jira through the Atlassian MCP. Use when the user wants to push epics/stories to Jira, publish the backlog to Jira, or reconcile the local backlog with a Jira project via the Atlassian MCP. Reads openspec/backlog/epics/ and openspec/backlog/stories/ produced by the planning chain.
---

The planning chain leaves a backlog on disk — `openspec/backlog/epics/` and `openspec/backlog/stories/` — that a team executes in Jira. This skill reconciles the two. It is the pipeline's **publishing step**: it runs after `pte-openspec-bdd-tests`, reads the epics and story cards, and mirrors them into a Jira project through the Atlassian MCP.

`ownership` is the governing word. Sync is not a direction — it is a set of fields, each with **exactly one owner**. Local owns the *content* it authored; Jira owns the *workflow* state a team adds. Every field flows from its owner to the other side and never the reverse, so a re-run is idempotent and never clobbers either side's work. Get the owner right and the rest is bookkeeping.

Shared pipeline conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, and diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md); read it before writing. A complete worked reconcile is in [`EXAMPLE.md`](EXAMPLE.md).

## Role in the pipeline

Input is the **artifact set** the planning chain produced: `openspec/backlog/epics/<epic-slug>.md` and `openspec/backlog/stories/<epic-slug>/<story-slug>.md`. This skill writes to **two places**: to Jira (create/update issues) and back into the local cards (a `## Jira szinkron` block — see below). It authors no epics or stories itself; if either is missing it stops and offers the chain step that produces them.

The Atlassian MCP must be connected first. It is provided by the `atlassian@claude-plugins-official` plugin (enabled in `.claude/settings.json`), authenticated once by the user via `/mcp` OAuth. If its tools are absent, stop and say so — this skill does not configure or authenticate it.

## Field ownership — the model the skill runs on

One owner per field. Push what local owns; pull what Jira owns. Never cross a field against its owner.

| Field | Owner | Direction |
|-------|-------|-----------|
| Summary (title), Description (User story + Kontextus), Elfogadási kritériumok, embedded BDD (Gherkin), labels (trace ID), issue type, epic↔story `parent` | **Local** | Local → Jira (push, overwrite) |
| Status, assignee, sprint, estimate (when the team sets it), comments, Jira key/URL | **Jira** | Jira → Local (pull into the card's `## Jira szinkron` block) |

The consequence that matters: **never** overwrite a locally-owned field from Jira, and **never** overwrite a Jira-owned workflow field from local. Status especially — the team's board owns it; this skill reads it, it does not set it.

## Jira mapping

| Local artifact | Jira |
|----------------|------|
| `openspec/backlog/epics/<epic-slug>.md` | issue type **Epic** |
| `openspec/backlog/stories/<epic-slug>/<story-slug>.md` | issue type **Story**, `parent` = the epic's Jira key |
| `EPIC-…` / `STORY-…` trace ID | label `trace:EPIC-…` / `trace:STORY-…` |
| card **Cím** title | Summary |
| card **User story** + **Kontextus** (+ **Elfogadási kritériumok**, **BDD teszt**) | Description |

The Description push carries **only** those content sections. It **excludes the `## Jira szinkron` block** — that block is a local-only record of Jira-owned state; pushing it back into the issue's own Description would be circular (writing Jira's key/status into Jira). The block lives on disk in the card, never in the ticket.

Because a story's `parent` needs its epic's Jira key, **process epics before stories**: resolve/create every epic first, record its key, then create stories against it.

**Epic-less stories** (a card whose parent `EPIC-…` line is empty — minted by `pte-openspec-to-stories`'s epic-less mode) have no epic on disk to parent them. Give them a single **standalone collector epic** as their Jira parent so they are never orphaned: resolve it by the reserved trace label `trace:EPIC-standalone` (create it once if absent — a plain Epic titled e.g. *Önálló változtatások / Standalone changes*), then set every epic-less story's `parent` to that collector's key. All the normal ownership and matching rules still apply — only the parent resolution differs.

## Matching — how a card finds its issue (idempotency)

Resolve each artifact to its issue in this order; the first hit wins:

1. **Recorded key** — the `Jira kulcs` in the card's `## Jira szinkron` block. Authoritative when present.
2. **Trace label** — `searchJiraIssuesUsingJql` with `labels = "trace:STORY-…"` (or `trace:EPIC-…`). The fallback when a card carries no recorded key yet.
3. **Create** — no match: `createJiraIssue`, then record the returned key back into the card (step below).

The trace label is what makes the reconcile idempotent across machines and fresh clones: even a card with no recorded key re-binds to its existing issue instead of duplicating it.

## The `## Jira szinkron` block — where the pull lands

After resolving (or creating) an issue, write Jira-owned fields back into the artifact as a `## Jira szinkron` section. Create it if absent; if present, replace only this section and leave every other section of the card untouched (diff-don't-clobber, per `CONVENTIONS.md`). Hungarian content, keys/URLs verbatim:

```markdown
## Jira szinkron
- Jira kulcs: PROJ-123
- Issue típus: Story
- Státusz: In Progress
- URL: https://<site>.atlassian.net/browse/PROJ-123
- Utolsó szinkron: <ISO-8601 időbélyeg>
```

This block is the durable record of match (1) and carries the Jira-owned status back onto disk so the backlog and the board agree.

## cloudId and project resolution

- Every Jira tool call needs a `cloudId`. Call `getAccessibleAtlassianResources` first (then `getVisibleJiraProjects` as needed) and thread the id through every call.
- The target project key is not hardcoded: read it from a `## Jira szinkron` block if one already names the project, else ask with **AskUserQuestion**. Confirm the project's Epic/Story issue types with `getJiraProjectIssueTypesMetadata` before creating — issue-type names vary per project.

## Atlassian MCP tools

- Read: `getAccessibleAtlassianResources`, `getVisibleJiraProjects`, `getJiraProjectIssueTypesMetadata`, `getJiraIssue`, `lookupJiraAccountId`, `searchJiraIssuesUsingJql`, `getTransitionsForJiraIssue`.
- Write: `createJiraIssue`, `editJiraIssue`, `addCommentToJiraIssue`, `transitionJiraIssue`.

## Run safety

Default to a **dry-run**: resolve every artifact and print the plan — what would be created, what updated, what pulled back — as trace ID → Jira key rows, **before any write**. Ask for confirmation with **AskUserQuestion** before the first `createJiraIssue` / `editJiraIssue`. Never open a status `transitionJiraIssue` unless the user explicitly asks — status is Jira-owned. On completion, print the same rows with their outcome (created / updated / pulled).

## Steps

Copy this checklist and tick each item — the verify step is exhaustive, not a glance:

```
- [ ] 1. Atlassian MCP reachable; cloudId + target project resolved (issue types confirmed)
- [ ] 2. Epic + story artifact set enumerated; missing-artifact fallback flagged
- [ ] 3. Each artifact resolved to an issue (recorded key → trace label → create) — dry-run plan printed
- [ ] 4. Writes confirmed by the user
- [ ] 5. Epics pushed first (keys recorded), then stories pushed with parent = epic key
- [ ] 6. Jira-owned fields pulled into each card's ## Jira szinkron block (diffed, not clobbered)
- [ ] 7. Every artifact reconciled 1:1 (no duplicate issues, no orphan stories); outcome rows reported
```

1. **Reach Jira, resolve context.** Confirm the Atlassian MCP tools exist (if not, stop — the server is not wired/authenticated). `getAccessibleAtlassianResources` → `cloudId`; resolve the project key (recorded block or AskUserQuestion); `getJiraProjectIssueTypesMetadata` → the project's Epic/Story issue-type ids. Completion: cloudId, project, and issue-type ids in hand.

2. **Enumerate artifacts.** List every `openspec/backlog/epics/<epic-slug>.md` and `openspec/backlog/stories/<epic-slug>/<story-slug>.md`, each with its trace ID from the card. If epics or stories are missing, stop and offer the producing step (`pte-openspec-to-epics` / `pte-openspec-to-stories` / `pte-openspec-bdd-tests`) rather than syncing a partial backlog. Completion: the full artifact set (or the flagged fallback) is named.

3. **Resolve + plan (dry-run).** For each artifact, find its issue by recorded key → trace label → (would-create). Print the plan as trace ID → Jira key (or "NEW") with the action per field owner. Completion: every artifact has a resolved action; the plan is printed and nothing is written yet.

4. **Confirm writes.** AskUserQuestion before the first write. Completion: the user has approved (or scoped down) the write set.

5. **Push local-owned fields.** Epics first: create/update each Epic, record its key; if any epic-less story is in the set, resolve/create the standalone collector epic (`trace:EPIC-standalone`) too. Then stories: create/update each Story with `parent` = its epic's Jira key (or the collector's key for epic-less stories), `trace:…` label, and Summary/Description from the card's owned sections. Never touch Jira-owned fields (status/assignee/sprint). Completion: every artifact has a live issue; every story's parent is set.

6. **Pull Jira-owned fields.** For each issue, read status/assignee/sprint/key/URL and write the `## Jira szinkron` block into its card — replace only that section, diff before overwriting. Completion: every card's block reflects the issue's current Jira-owned state.

7. **Verify exhaustively.** Every epic and story maps to exactly one issue (no duplicates from a missed match); every story's `parent` resolves to its epic's issue; every card carries a `## Jira szinkron` block with a real key; no locally-owned field was pulled from Jira and no Jira-owned field was pushed from local. Report any artifact you could not reconcile cleanly rather than guessing.
