---
name: pte-openspec-jira-sync
description: Sync the pipeline's epics and story cards to Jira through the Atlassian MCP. Use when the user wants to push epics/stories to Jira, publish the backlog to Jira, or reconcile the local backlog with a Jira project via the Atlassian MCP. Reads openspec/backlog/epics/ and openspec/backlog/stories/ produced by the planning chain.
---

The planning chain leaves a backlog on disk — `openspec/backlog/epics/` and `openspec/backlog/stories/` — that a team executes in Jira. This skill reconciles the two. It is the pipeline's **publishing step**: it runs after `pte-openspec-bdd-tests`, reads the epics and story cards, and mirrors them into a Jira project through the Atlassian MCP.

`ownership` is the governing word. Sync is not a direction — it is a set of fields, each with **exactly one owner**. Local owns the *content* it authored; Jira owns the *workflow* state a team adds. Every field flows from its owner to the other side and never the reverse, so a re-run is idempotent and never clobbers either side's work. Get the owner right and the rest is bookkeeping.

Shared pipeline conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, and diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md); read it before writing. A complete worked reconcile is in [`EXAMPLE.md`](EXAMPLE.md).

## Role in the pipeline

Input is the **artifact set** the planning chain produced: `openspec/backlog/epics/EPIC-<epic-slug>.md` and `openspec/backlog/stories/<epic-slug>/<story-slug>.md` (the story directory's `<epic-slug>` is the epic filename **minus** its `EPIC-` prefix). This skill writes to **two places**: to Jira (create/update issues) and back into the local cards (a `## Jira szinkron` block — see below). It authors no epics or stories itself; if either is missing it stops and offers the chain step that produces them.

The Atlassian MCP must be connected first. It is provided by the `atlassian@claude-plugins-official` plugin (enabled in `.claude/settings.json`), authenticated once by the user via `/mcp` OAuth. If its tools are absent, stop and say so — this skill does not configure or authenticate it.

## Field ownership — the model the skill runs on

One owner per field. Push what local owns; pull what Jira owns. Never cross a field against its owner.

| Field | Owner | Direction |
|-------|-------|-----------|
| Summary (H1 title, minus its ` · TRACE-ID` suffix), Description (the **full card body** below the H1, minus the `## Jira szinkron` block), labels (trace ID), issue type, epic↔story `parent` | **Local** | Local → Jira (push, overwrite) |
| Status, assignee, sprint, estimate (when the team sets it), comments, Jira key/URL | **Jira** | Jira → Local (pull into the card's `## Jira szinkron` block) |

The consequence that matters: **never** overwrite a locally-owned field from Jira, and **never** overwrite a Jira-owned workflow field from local. Status especially — the team's board owns it; this skill reads it, it does not set it.

## Jira mapping

| Local artifact | Jira |
|----------------|------|
| `openspec/backlog/epics/EPIC-<epic-slug>.md` | issue type **Epic** |
| `openspec/backlog/stories/<epic-slug>/<story-slug>.md` | issue type **Story**, `parent` = the epic's Jira key |
| `EPIC-…` / `STORY-…` trace ID | label `trace:EPIC-…` / `trace:STORY-…` |
| card **Cím** (H1 title, minus its ` · TRACE-ID` suffix) | Summary |
| the **full card body below the H1**, minus the `## Jira szinkron` block | Description |

The Description push carries the **full card body** — every section below the H1 title, verbatim (User story, Kontextus, Elfogadási kritériumok, BDD teszt, INVEST-ellenőrzés, DoR/DoD, Prioritás, Becslés, Függőségek és kockázatok, Forrás-hivatkozás — the whole card, whatever sections it has, not a chosen subset). It **excludes exactly two things**: the H1 title line (that maps to Summary), and the `## Jira szinkron` block (a local-only record of Jira-owned state — pushing it back into the issue's own Description would be circular). Slice the card from disk by those two boundaries and push that slice; do **not** reconstruct sections from memory — a verbatim file slice avoids transcription drift on re-push.

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

## Cross-reference backlinks (opt-in)

By default the only Jira key written into a card is its own, in the `## Jira szinkron` block. When the user wants the **cross-references inside the body** to carry live Jira links too — every `EPIC-…`/`STORY-…` a card mentions: the `**Epic:**` metadata line, an epic's `### STORY-…` headings, prose dependencies (*Függőségek és kockázatok*), and every trace-ID cell in the *Forrás-hivatkozás* table — run this as a final enrichment pass **after every key is assigned**:

- **Format:** keep the trace-ID text and append ` ([<KEY>](<browse-url>))` right after it — e.g. `EPIC-idoszak-lezaras ([INYO-21](https://<site>.atlassian.net/browse/INYO-21))`.
- **Idempotent:** skip any reference that already carries a link; never double-link. A re-run is a no-op.
- **Never linkify:** the card's own trace ID in the H1 title; the `@EPIC-…`/`@STORY-…` tags inside the ```gherkin block (they are Cucumber tags); anything in the `## Jira szinkron` block.
- After editing the cards, **re-push** each affected Description so Jira mirrors the linked body.
- **Caveat to state first:** this writes Jira keys into the source cards, coupling the backlog to one Jira instance. Idempotency still holds — the trace IDs stay verbatim, so matching (label + recorded key) is unaffected — but offer it, don't assume it.

## cloudId and project resolution

- Every Jira tool call needs a `cloudId`. Call `getAccessibleAtlassianResources` first (then `getVisibleJiraProjects` as needed) and thread the id through every call.
- The target project key is not hardcoded: read it from a `## Jira szinkron` block if one already names the project, else ask with **AskUserQuestion**. Confirm the project's Epic/Story issue types with `getJiraProjectIssueTypesMetadata` before creating — issue-type names vary per project **and are localized**: a Hungarian project names the Epic type **"Eposz"** (Story stays "Story"). Resolve the type by `untranslatedName` (`Epic`/`Story`) or `hierarchyLevel` (Epic = 1, Story = 0), then pass the project's actual display `name` to `createJiraIssue`'s `issueTypeName`.

## Atlassian MCP tools

- Read: `getAccessibleAtlassianResources`, `getVisibleJiraProjects`, `getJiraProjectIssueTypesMetadata`, `getJiraIssue`, `lookupJiraAccountId`, `searchJiraIssuesUsingJql`, `getTransitionsForJiraIssue`.
- Write: `createJiraIssue`, `editJiraIssue`, `addCommentToJiraIssue`, `transitionJiraIssue`.

## Operational notes

- **JQL `labels` has no `LIKE`/wildcard.** To find existing trace-labelled issues, enumerate the exact labels in one `labels IN ("trace:EPIC-…","trace:STORY-…", …)` set (or one query per label). `labels LIKE "trace:*"` is rejected by JQL.
- **Large MCP responses overflow context.** `getVisibleJiraProjects` on a busy site can exceed the tool-result limit and be spilled to a file. Parse it from that file (jq/python) or narrow it with `searchString` — don't read the whole blob inline.
- **Re-push Descriptions from the file, not from memory.** When updating an existing issue's Description, read the card and push a verbatim slice (body minus H1 minus `## Jira szinkron`); reconstructing the body by hand risks transcription typos and a needless corrective round-trip.

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
- [ ] 6b. (opt-in) Cross-reference backlinks written into card bodies; affected Descriptions re-pushed
- [ ] 7. Every artifact reconciled 1:1 (no duplicate issues, no orphan stories); outcome rows reported
```

1. **Reach Jira, resolve context.** Confirm the Atlassian MCP tools exist (if not, stop — the server is not wired/authenticated). `getAccessibleAtlassianResources` → `cloudId`; resolve the project key (recorded block or AskUserQuestion); `getJiraProjectIssueTypesMetadata` → the project's Epic/Story issue-type ids. Completion: cloudId, project, and issue-type ids in hand.

2. **Enumerate artifacts.** List every `openspec/backlog/epics/EPIC-<epic-slug>.md` and `openspec/backlog/stories/<epic-slug>/<story-slug>.md`, each with its trace ID from the card (pair a story directory to its epic by the epic filename minus its `EPIC-` prefix). If epics or stories are missing, stop and offer the producing step (`pte-openspec-to-epics` / `pte-openspec-to-stories` / `pte-openspec-bdd-tests`) rather than syncing a partial backlog. Completion: the full artifact set (or the flagged fallback) is named.

3. **Resolve + plan (dry-run).** For each artifact, find its issue by recorded key → trace label → (would-create). Print the plan as trace ID → Jira key (or "NEW") with the action per field owner. Completion: every artifact has a resolved action; the plan is printed and nothing is written yet.

4. **Confirm writes.** AskUserQuestion before the first write. Completion: the user has approved (or scoped down) the write set.

5. **Push local-owned fields.** Epics first: create/update each Epic, record its key; if any epic-less story is in the set, resolve/create the standalone collector epic (`trace:EPIC-standalone`) too. Then stories: create/update each Story with `parent` = its epic's Jira key (or the collector's key for epic-less stories), `trace:…` label, Summary from the H1 title (minus its ` · TRACE-ID` suffix), and Description from the **full card body** (minus the H1 and the `## Jira szinkron` block — see *Jira mapping*). Never touch Jira-owned fields (status/assignee/sprint). Completion: every artifact has a live issue; every story's parent is set.

6. **Pull Jira-owned fields.** For each issue, read status/assignee/sprint/key/URL and write the `## Jira szinkron` block into its card — replace only that section, diff before overwriting. Completion: every card's block reflects the issue's current Jira-owned state.

6b. **Cross-reference backlinks (opt-in).** Only when the user asks for it: with every key now assigned, linkify the body cross-references per *Cross-reference backlinks* above (idempotent, exact format, skip the H1/gherkin-tags/szinkron block), then re-push each affected Description. Completion: every card's body cross-reference carries its `([<KEY>](url))` link and Jira mirrors it.

7. **Verify exhaustively.** Every epic and story maps to exactly one issue (no duplicates from a missed match); every story's `parent` resolves to its epic's issue; every card carries a `## Jira szinkron` block with a real key; no locally-owned field was pulled from Jira and no Jira-owned field was pushed from local. Report any artifact you could not reconcile cleanly rather than guessing.
