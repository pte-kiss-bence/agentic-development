---
name: pte-openspec-jira-sync
description: Sync the pipeline's epics and story cards to Jira through the Atlassian MCP. Use when the user wants to publish or reconcile the local backlog (openspec/backlog/epics/ + stories/) with a Jira project via the Atlassian MCP.
---

The planning chain leaves a backlog on disk — `openspec/backlog/epics/` and `openspec/backlog/stories/` — that a team executes in Jira. This skill reconciles the two. It is the pipeline's **publishing step**: it runs after `pte-openspec-bdd-tests`, reads the epics and story cards, and mirrors them into a Jira project through the Atlassian MCP.

`ownership` is the governing word. Sync is not a direction — it is a set of fields, each with **exactly one owner**. Local owns the *content* it authored; Jira owns the *workflow* state a team adds. Every field flows from its owner to the other side and never the reverse, so a re-run is idempotent and never clobbers either side's work. Get the owner right and the rest is bookkeeping.

Shared pipeline conventions — output language, trace-ID grammar, the *Forrás-spec hivatkozás* map contract, source-spec resolution, and diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md); read it before writing. A complete worked reconcile is in [`EXAMPLE.md`](EXAMPLE.md).

## Role in the pipeline

Input is the **artifact set** the planning chain produced: `openspec/backlog/epics/EPIC-<epic-slug>.md` and `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md` (the story directory's `<epic-slug>` is the epic filename **minus** its `EPIC-` prefix). This skill writes to **two places**: to Jira (create/update issues) and back into the local cards (a `## Jira szinkron` block — see below). It authors no epics or stories itself; if either is missing it stops and offers the chain step that produces them.

The Atlassian MCP must be connected first. It is provided by the `atlassian@claude-plugins-official` plugin (enabled in `.claude/settings.json`), authenticated once by the user via `/mcp` OAuth. If its tools are absent, stop and say so — this skill does not configure or authenticate it.

## Field ownership — the model the skill runs on

One owner per field. Push what local owns; pull what Jira owns. Never cross a field against its owner.

| Field | Owner | Direction |
|-------|-------|-----------|
| Summary (H1 title, minus its ` · TRACE-ID` suffix), Description (the **Jira-visible card body** below the H1, minus the `## Jira szinkron` block **and** the `<!-- pipeline-only -->` block), labels (trace ID), issue type, epic↔story `parent`, **Story points** (the SP from the card's `## Estimation`) | **Local** | Local → Jira (push, overwrite) |
| Status, assignee, sprint, comments, Jira key/URL | **Jira** | Jira → Local (pull into the card's `## Jira szinkron` block) |

The consequence that matters: **never** overwrite a locally-owned field from Jira, and **never** overwrite a Jira-owned workflow field from local. Status especially — the team's board owns it; this skill reads it, it does not set it.

**Story points is now local-owned** — a deliberate flip from the old "estimate is Jira-owned" model. The pipeline's `## Estimation` (see *Estimation* in `CONVENTIONS.md`) is the AI-authored **reference base** the team relies on, so the SP flows **Local → Jira** into the structured *Story points* field; it is not pulled back. (The PERT base — `Eβ`/`σ` — rides along inside the Description as part of the card body; only the SP maps to the structured field. The downstream manager multipliers are applied off-Jira and never sync.)

## Jira mapping

| Local artifact | Jira |
|----------------|------|
| `openspec/backlog/epics/EPIC-<epic-slug>.md` | issue type **Epic** |
| `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md` | issue type **Story**, `parent` = the epic's Jira key |
| `EPIC-…` / `STORY-…` trace ID | label `trace:EPIC-…` / `trace:STORY-…` |
| card **Cím** (H1 title, minus its ` · TRACE-ID` suffix) | Summary |
| the **Jira-visible card body below the H1**, minus the `## Jira szinkron` block and the `<!-- pipeline-only -->` block | Description |
| story card `## Estimation` Story points (the bare SP integer) | the issue's structured **Story points** field |

The Story-points field is a **custom field** whose id varies per project (e.g. `customfield_10016`) and whose name is often localized. Resolve it once via the project's field metadata (`getJiraProjectIssueTypesMetadata` / field list) rather than hardcoding an id; if the project has no Story-points field, skip the structured push and say so (the SP still travels inside the Description as part of `## Estimation`). Only the **story** SP is pushed to the field; an epic's `## Estimation` is a rollup that rides along in its Description only.

The Description push carries the **Jira-visible card body** — the sections below the H1 title, verbatim (epic: Description, Persona, E2E Scenario, Problem / Solution, Cross-cutting Concerns, MVP and Out of Scope, Success metrics, Risks and Dependencies, High Level Acceptance Criteria; story: Description, Context, BDD Test, Risks and Dependencies — the whole visible card, whatever sections it has, not a chosen subset). It **excludes exactly three things**: the H1 title line (that maps to Summary); the `## Jira szinkron` block (a local-only record of Jira-owned state — pushing it back into the issue's own Description would be circular); and the **pipeline-only block** — everything between `<!-- pipeline-only:start -->` and `<!-- pipeline-only:end -->`, markers inclusive (the epic's *User story-k* + *Forrás-spec hivatkozás* map, or an epic-less card's self-carried *Forrás-hivatkozás* row — pipeline-internal, never a Jira field; see `CONVENTIONS.md`). Slice the card from disk by those three boundaries and push that slice; do **not** reconstruct sections from memory — a verbatim file slice avoids transcription drift on re-push.

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

By default the only Jira key written into a card is its own, in the `## Jira szinkron` block. When the user wants the **cross-references inside the body** to carry live Jira links too, run this as a final enrichment pass **after every key is assigned**. Link **every** `EPIC-…`/`STORY-…` a card mentions in a **Jira-visible** section — do not work from an allow-list of sections (that is how references get missed). Covered spots include, but are **not limited to**: the `**Epic:**` metadata line, *MVP and Out of Scope* "lásd …" pointers, *E2E Scenario*/*Description*/*Context*/*Success metrics* prose, and *Risks and Dependencies* bullets. A reference counts whether or not it is wrapped in backticks (`` `EPIC-x` `` and plain `EPIC-x` both get the link). Do **not** linkify inside the pipeline-only block — it is stripped from the Jira Description, so its trace IDs (the *Forrás-spec hivatkozás* / *Forrás-hivatkozás* map) stay bare for the downstream skills to match on.

- **Format:** keep the trace-ID text (and its backticks, if any) and append ` ([<KEY>](<browse-url>))` right after it — e.g. `` `EPIC-idoszak-lezaras` ([INYO-21](https://<site>.atlassian.net/browse/INYO-21)) `` or `EPIC-idoszak-lezaras ([INYO-21](https://<site>.atlassian.net/browse/INYO-21))`.
- **Coverage check:** after linking, scan every card for any `EPIC-…`/`STORY-…` **not** followed by a `([INYO-…])` link (allowing an optional closing backtick between them); the only hits left should be the exempt cases below. Also verify each link's key matches the trace ID's real issue and the URL is `…/browse/<key>` — a wrong-target link is worse than a missing one.
- **Idempotent:** skip any reference that already carries a link; never double-link. A re-run is a no-op.
- **Never linkify (the only exemptions):** the card's own trace ID in the H1 title (self-identity, already in `## Jira szinkron`); the `@EPIC-…`/`@STORY-…` tags and any IDs inside the ```gherkin block (Cucumber tags / test text); anything in the `## Jira szinkron` block; and anything inside the `<!-- pipeline-only -->` block (stripped from Jira, matched on bare IDs).
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
- **Re-push Descriptions from the file, not from memory.** When updating an existing issue's Description, read the card and push a verbatim slice (body minus H1, minus `## Jira szinkron`, minus the `<!-- pipeline-only -->` block); reconstructing the body by hand risks transcription typos and a needless corrective round-trip.

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

2. **Enumerate artifacts.** List every `openspec/backlog/epics/EPIC-<epic-slug>.md` and `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md`, each with its trace ID from the card (pair a story directory to its epic by the epic filename minus its `EPIC-` prefix). If epics or stories are missing, stop and offer the producing step (`pte-openspec-to-epics` / `pte-openspec-to-stories` / `pte-openspec-bdd-tests`) rather than syncing a partial backlog. Completion: the full artifact set (or the flagged fallback) is named.

3. **Resolve + plan (dry-run).** For each artifact, find its issue by recorded key → trace label → (would-create). Print the plan as trace ID → Jira key (or "NEW") with the action per field owner. Completion: every artifact has a resolved action; the plan is printed and nothing is written yet.

4. **Confirm writes.** AskUserQuestion before the first write. Completion: the user has approved (or scoped down) the write set.

5. **Push local-owned fields.** Epics first: create/update each Epic, record its key; if any epic-less story is in the set, resolve/create the standalone collector epic (`trace:EPIC-standalone`) too. Then stories: create/update each Story with `parent` = its epic's Jira key (or the collector's key for epic-less stories), `trace:…` label, Summary from the H1 title (minus its ` · TRACE-ID` suffix), Description from the **Jira-visible card body** (minus the H1, the `## Jira szinkron` block, and the `<!-- pipeline-only -->` block — see *Jira mapping*), and the **Story points** field from the card's `## Estimation` SP (resolve the custom field id first; skip the structured push if the project has no Story-points field). Never touch Jira-owned fields (status/assignee/sprint). Completion: every artifact has a live issue; every story's parent is set; every story's SP pushed (or skipped-with-reason).

6. **Pull Jira-owned fields.** For each issue, read status/assignee/sprint/key/URL and write the `## Jira szinkron` block into its card — replace only that section, diff before overwriting. Completion: every card's block reflects the issue's current Jira-owned state.

6b. **Cross-reference backlinks (opt-in).** Only when the user asks for it: with every key now assigned, linkify the body cross-references per *Cross-reference backlinks* above (idempotent, exact format, skip the H1/gherkin-tags/szinkron block), then re-push each affected Description. Completion: every card's body cross-reference carries its `([<KEY>](url))` link and Jira mirrors it.

7. **Verify exhaustively.** Every epic and story maps to exactly one issue (no duplicates from a missed match); every story's `parent` resolves to its epic's issue; every card carries a `## Jira szinkron` block with a real key; no locally-owned field was pulled from Jira and no Jira-owned field was pushed from local. Report any artifact you could not reconcile cleanly rather than guessing.
