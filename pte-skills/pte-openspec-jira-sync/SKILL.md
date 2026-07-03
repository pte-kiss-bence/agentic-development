---
name: pte-openspec-jira-sync
description: Sync the pipeline's epics and story cards to Jira through the Atlassian MCP. Use when the user wants to publish or reconcile the local backlog (openspec/backlog/epics/ + stories/) with a Jira project via the Atlassian MCP.
---

The planning chain leaves a backlog on disk — `openspec/backlog/epics/` and `openspec/backlog/stories/` — that a team executes in Jira. This skill reconciles the two. It is the pipeline's **publishing step**: it runs after `pte-openspec-bdd-tests`, reads the epics and story cards, and mirrors them into a Jira project through the Atlassian MCP.

`ownership` is the governing word. Sync is not a direction — it is a set of fields, each with **exactly one owner**. Local owns the *content* it authored; Jira owns the *workflow* state a team adds. Every field flows from its owner to the other side and never the reverse, so a re-run is idempotent and never clobbers either side's work. Get the owner right and the rest is bookkeeping.

Shared pipeline conventions — output language, trace-ID grammar, the *Source Spec Reference* map contract, source-spec resolution, and diff-don't-clobber — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md); read it before writing. A complete worked reconcile is in [`EXAMPLE.md`](EXAMPLE.md).

## Role in the pipeline

Input is the **artifact set** the planning chain produced: `openspec/backlog/epics/EPIC-<epic-slug>.md` and `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md` (the story directory's `<epic-slug>` is the epic filename **minus** its `EPIC-` prefix). This skill writes to **two places**: to Jira (create/update issues) and back into the local cards (a `## Jira Sync` block — see below). It authors no epics or stories itself; if either is missing it stops and offers the chain step that produces them.

The Atlassian MCP must be connected first. It is provided by the `atlassian@claude-plugins-official` plugin (enabled in `.claude/settings.json`), authenticated once by the user via `/mcp` OAuth. If its tools are absent, stop and say so — this skill does not configure or authenticate it.

## Field ownership — the model the skill runs on

One owner per field. Push what local owns; pull what Jira owns. Never cross a field against its owner.

| Field | Owner | Direction |
|-------|-------|-----------|
| Summary (H1 title, minus its ` · TRACE-ID` suffix), Description (the **Jira-visible card body** below the H1, minus the `## Jira Sync` block **and** the `<!-- pipeline-only -->` block), labels (trace ID), issue type, epic↔story `parent`, **Story points** (the SP from the card's `## Estimation`), **Original estimate** (the `Becsült munkaóra` = `Eβ` ideal hours) | **Local** | Local → Jira (push, overwrite) |
| **Dependency issue links** (the Blocks / Relates links built from the card's `## Dependency Edges` block) | **Local** | Local → Jira (push, **additive** — created, never deleted; see *Dependency issue links*) |
| Status, assignee, sprint, comments, Jira key/URL | **Jira** | Jira → Local (pull into the card's `## Jira Sync` block) |

The consequence that matters: **never** overwrite a locally-owned field from Jira, and **never** overwrite a Jira-owned workflow field from local. Status especially — the team's board owns it; this skill reads it, it does not set it.

**Story points and Original estimate are now local-owned** — a deliberate flip from the old "estimate is Jira-owned" model. The pipeline's `## Estimation` (see *Estimation* in `CONVENTIONS.md`) is the AI-authored **reference base** the team relies on, so it flows **Local → Jira** into two structured fields (never pulled back): the **Story points** field gets the SP, and the **Original estimate** (time-tracking) field gets the `Becsült munkaóra` — the `Eβ` ideal engineer-hours — pushed as `<Eβ>h` (e.g. `11h`). (A story card's `## Estimation` renders three stacked bullets — the inline `3-Points becslés` (`O`/`M`/`P`/`Eβ`/`σ`), `Becsült munkaóra:` (`Eβ` ó + ideal days), and `Story Point:` (`<N>`, bare integer); an epic drops the `3-Points` bullet. The whole `## Estimation` section lives **inside the `<!-- pipeline-only -->` fence**, so it is **stripped from the pushed Description** — the sync reads it from that fenced block on disk, never from the Description.) **Both** stories and epics push these two structured fields: a story pushes its own SP + `Becsült munkaóra`, and an **epic pushes its rollup** — the summed `Story Point` into the Story-points field and the summed `Becsült munkaóra` into Original estimate (`<ΣEβ>h`). The downstream manager multipliers are applied off-Jira and never sync.

## Jira mapping

| Local artifact | Jira |
|----------------|------|
| `openspec/backlog/epics/EPIC-<epic-slug>.md` | issue type **Epic** |
| `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md` | issue type **Story**, `parent` = the epic's Jira key |
| `EPIC-…` / `STORY-…` trace ID | label `trace:EPIC-…` / `trace:STORY-…` |
| card **Cím** (H1 title, minus its ` · TRACE-ID` suffix) | Summary |
| the **Jira-visible card body below the H1**, minus the `## Jira Sync` block and the `<!-- pipeline-only -->` block | Description |
| story card `## Estimation` SP — the integer on the `- **Story Point:** <N>` line | the issue's structured **Story points** field |
| story card `## Estimation` `Becsült munkaóra` — the `Eβ` hours on the `- **Becsült munkaóra:** <Eβ> ó …` line | the issue's **Original estimate** (time-tracking), pushed as `<Eβ>h` via `additional_fields: {"timetracking": {"originalEstimate": "<Eβ>h"}}` |

The Story-points field is a **custom field** whose id varies per project (e.g. `customfield_10016`) and whose name is often localized. Resolve it once via the project's field metadata (`getJiraProjectIssueTypesMetadata` / field list) rather than hardcoding an id; if the project has no Story-points field, skip the structured push and say so. **Both** stories and epics push the Story-points + Original-estimate fields: the epic pushes its **rollup** sums (`ΣSP`, `ΣEβ`h), the story its own values. The source `## Estimation` section is **pipeline-only** (inside the fence, stripped from the Description) — read the numbers from that fenced block, not from the Description.

The Description push carries the **Jira-visible card body** — the sections below the H1 title, verbatim (epic: Epic, Persona, E2E Scenario, Problem / Solution, Cross-cutting Concerns, MVP and Out of Scope, Success metrics, Risks and Dependencies, High Level Acceptance Criteria; story: User Story, Context, BDD Test, Risks and Dependencies — the whole visible card, whatever sections it has, not a chosen subset). It **excludes exactly three things**: the H1 title line (that maps to Summary); the `## Jira Sync` block (a local-only record of Jira-owned state — pushing it back into the issue's own Description would be circular); and the **pipeline-only block** — everything between `<!-- pipeline-only:start -->` and `<!-- pipeline-only:end -->`, markers inclusive (the epic's *User Stories* + *Source Spec Reference* map, or an epic-less card's self-carried *Source Reference* row — pipeline-internal, never a Jira field; see `CONVENTIONS.md`). Slice the card from disk by those three boundaries and push that slice; do **not** reconstruct sections from memory — a verbatim file slice avoids transcription drift on re-push.

Because a story's `parent` needs its epic's Jira key, **process epics before stories**: resolve/create every epic first, record its key, then create stories against it.

**Epic-less stories** (a card whose parent `EPIC-…` line is empty — minted by `pte-openspec-to-stories`'s epic-less mode) have no epic on disk to parent them. Give them a single **standalone collector epic** as their Jira parent so they are never orphaned: resolve it by the reserved trace label `trace:EPIC-standalone` (create it once if absent — a plain Epic titled e.g. *Önálló változtatások / Standalone changes*), then set every epic-less story's `parent` to that collector's key. All the normal ownership and matching rules still apply — only the parent resolution differs.

## Matching — how a card finds its issue (idempotency)

Resolve each artifact to its issue in this order; the first hit wins:

1. **Recorded key** — the `Jira key` in the card's `## Jira Sync` block. Authoritative when present.
2. **Trace label** — `searchJiraIssuesUsingJql` with `labels = "trace:STORY-…"` (or `trace:EPIC-…`). The fallback when a card carries no recorded key yet.
3. **Create** — no match: `createJiraIssue`, then record the returned key back into the card (step below).

The trace label is what makes the reconcile idempotent across machines and fresh clones: even a card with no recorded key re-binds to its existing issue instead of duplicating it.

## The `## Jira Sync` block — where the pull lands

After resolving (or creating) an issue, write Jira-owned fields back into the artifact as a `## Jira Sync` section. Create it if absent; if present, replace only this section and leave every other section of the card untouched (diff-don't-clobber, per `CONVENTIONS.md`). English field labels; the Status value plus keys/URLs are verbatim from Jira:

```markdown
## Jira Sync
- Jira key: PROJ-123
- Issue type: Story
- Status: In Progress
- URL: https://<site>.atlassian.net/browse/PROJ-123
- Last sync: <ISO-8601 időbélyeg>
```

This block is the durable record of match (1) and carries the Jira-owned status back onto disk so the backlog and the board agree.

## Change-summary comment on update

When a sync **updates an already-existing issue** and at least one locally-owned field actually changed, post **one** Jira comment summarizing what changed — an audit trail on the issue itself so a team watching the board sees why it moved. This is scoped tightly:

- **Update only, never create.** A freshly created issue gets **no** summary comment — the whole issue is the record. The comment fires only on the update path (issue resolved via recorded key or trace label).
- **Only when something changed.** Diff each locally-owned field (Summary, Description, labels, `parent`, Story points, Original estimate) against the issue's current value **before** writing. If every field is identical — a no-op re-sync — post **nothing**; the run stays idempotent (no comment spam on repeat runs). The comment is gated on a real diff, not on the update path being taken.
- **One comment per issue per run**, listing only the fields that changed. Name each changed field, and give a terse old → new for the short structured ones (Story points, Original estimate, Summary, parent, labels). For Description (long free text) state *that* it changed, not a full text diff — e.g. `Leírás frissítve`. Keep it to the fields that moved; don't list unchanged ones.
- **Hungarian, dated.** Comment body in Hungarian (per `CONVENTIONS.md` output language — Jira issue content follows the artifact language), led by an ISO-8601 timestamp and a marker line so the audit entries are scannable. Shape:

  ```markdown
  🔄 Pipeline szinkron — <ISO-8601 időbélyeg>
  Változott mezők:
  - Story points: 5 → 8
  - Original estimate: 11h → 18h
  - Leírás frissítve
  ```

- **Ownership note.** This does **not** break the "comments are Jira-owned" rule: the ownership table governs *reading* comments back into the card (which the skill still never does — it does not pull comments to disk). The change-summary comment is a **write-only audit note** the skill appends on update; it is never read back, so no field is crossed against its owner.

Post the comment with `addCommentToJiraIssue`, **after** the field push for that issue succeeds (so the summary reflects what actually landed), and before moving to the next artifact.

## Cross-reference backlinks (opt-in)

By default the only Jira key written into a card is its own, in the `## Jira Sync` block. When the user wants the **cross-references inside the body** to carry live Jira links too, run this as a final enrichment pass **after every key is assigned**. Link **every** `EPIC-…`/`STORY-…` a card mentions in a **Jira-visible** section — do not work from an allow-list of sections (that is how references get missed). Covered spots include, but are **not limited to**: *MVP and Out of Scope* "lásd …" pointers, *E2E Scenario*/*Epic*/*User Story*/*Context*/*Success metrics* prose, and *Risks and Dependencies* bullets. (The `Parent epic:` line is **not** a spot — it lives inside the pipeline-only fence, which is never linkified.) A reference counts whether or not it is wrapped in backticks (`` `EPIC-x` `` and plain `EPIC-x` both get the link). Do **not** linkify inside the pipeline-only block — it is stripped from the Jira Description, so its trace IDs (the *Source Spec Reference* / *Source Reference* map) stay bare for the downstream skills to match on.

- **Format:** keep the trace-ID text (and its backticks, if any) and append ` ([<KEY>](<browse-url>))` right after it — e.g. `` `EPIC-idoszak-lezaras` ([INYO-21](https://<site>.atlassian.net/browse/INYO-21)) `` or `EPIC-idoszak-lezaras ([INYO-21](https://<site>.atlassian.net/browse/INYO-21))`.
- **Coverage check:** after linking, scan every card for any `EPIC-…`/`STORY-…` **not** followed by a `([INYO-…])` link (allowing an optional closing backtick between them); the only hits left should be the exempt cases below. Also verify each link's key matches the trace ID's real issue and the URL is `…/browse/<key>` — a wrong-target link is worse than a missing one.
- **Idempotent:** skip any reference that already carries a link; never double-link. A re-run is a no-op.
- **Never linkify (the only exemptions):** the card's own trace ID in the H1 title (self-identity, already in `## Jira Sync`); the `@EPIC-…`/`@STORY-…` tags and any IDs inside the ```gherkin block (Cucumber tags / test text); anything in the `## Jira Sync` block; and anything inside the `<!-- pipeline-only -->` block (stripped from Jira, matched on bare IDs).
- After editing the cards, **re-push** each affected Description so Jira mirrors the linked body.
- **Caveat to state first:** this writes Jira keys into the source cards, coupling the backlog to one Jira instance. Idempotency still holds — the trace IDs stay verbatim, so matching (label + recorded key) is unaffected — but offer it, don't assume it.

## Dependency issue links (default on)

With every issue key assigned (step 5), mirror the backlog's dependencies as **real Jira issue links**. The source is each card's **`## Dependency Edges`** block — the single source of truth per the [edge contract in `CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md#the-dependency-edges-block--the-edge-contract). Never read edges from the `## Risks and Dependencies` prose, and never from `DEPENDENCY_GRAPH.md` (that is a derived view). This runs **by default**, gated by the run's existing write-confirmation (step 4) — it is **not** a separate opt-in like the cross-reference backlinks.

- **Link-type mapping**, resolved via `getIssueLinkTypes` (localization-tolerant):
  - `blocks: \`Y\`` on X → X **blocks** Y (the *Blocks* link type, outward from X).
  - `depends-on: \`Z\`` on X → Z **blocks** X (the same *Blocks* type, reversed direction).
  - `relates-to: \`W\`` → the *Relates* link type (symmetric).
  - `external: …` → **skipped** — no Jira issue exists for an external node.
- **Resolve the link type by its canonical name, not its display name** — a Hungarian site localizes them (*Blokkolja* / *Kapcsolódik*). Match on `getIssueLinkTypes`' stable/English name; if the project renamed or removed the type and it can't be resolved unambiguously, **AskUserQuestion** which link type to use (the same pattern as the localized issue-type resolution).
- **Direction is load-bearing.** Create each *Blocks* link with the correct inward/outward end so the board reads "X blocks Y" exactly as the edge does — a reversed link is worse than a missing one.
- **Idempotent + inverse-dedup.** Before creating, read the issue's existing `issuelinks` (`getJiraIssue`): never duplicate a link already present, and treat X-blocks-Y and Y-is-blocked-by-X as **one** link created once (the edge contract already deduped the inverse pair; hold that here too). A re-run creates nothing new.
- **Dangling target → warn and skip.** An edge whose target has no issue in the backlog (or no resolvable key) is warned and skipped — never link against a guessed key.
- **Additive + drift-report — never delete.** If a Jira dependency-link no longer matches any local edge (an edge removed on disk, or a link a team added by hand), **report it as drift** in the outcome and leave removal to a human. This keeps "never clobber the other side's work" intact — a team may link issues the backlog doesn't know about.
- Create with `createIssueLink`; report per issue how many links were **created / already-present / skipped (external or dangling) / drift**.

## cloudId and project resolution

- Every Jira tool call needs a `cloudId`. Call `getAccessibleAtlassianResources` first (then `getVisibleJiraProjects` as needed) and thread the id through every call.
- The target project key is not hardcoded: read it from a `## Jira Sync` block if one already names the project, else ask with **AskUserQuestion**. Confirm the project's Epic/Story issue types with `getJiraProjectIssueTypesMetadata` before creating — issue-type names vary per project **and are localized**: a Hungarian project names the Epic type **"Eposz"** (Story stays "Story"). Resolve the type by `untranslatedName` (`Epic`/`Story`) or `hierarchyLevel` (Epic = 1, Story = 0), then pass the project's actual display `name` to `createJiraIssue`'s `issueTypeName`.

### Epic parent — the level above Epic (project-container issue)

A Jira **project is not an issue**, so it is never itself a `parent`. But some projects (Premium sites with custom hierarchy) place Epics **under a higher-level issue** — a `Project`/`Initiative`-type container (`hierarchyLevel ≥ 2`) that represents the whole initiative. When the target project uses such a level, **parent each synced Epic under that project-container issue** so the synced epics sit where the project's existing epics already sit (do not leave them dangling at the top).

Resolve the container: `searchJiraIssuesUsingJql` for the project's above-Epic issues (e.g. `project = <KEY> AND issuetype = Project`, or check an existing epic's `parent`). Then:
- **exactly one** such container → use it as every synced Epic's `parent` (pass its key to `createJiraIssue`'s `parent`);
- **several** → **AskUserQuestion** which one;
- **none** (the project's top level *is* Epic — the common flat case) → the Epic has **no** parent.

Stories are unaffected — a story's `parent` is always its own Epic's key (or the standalone collector epic for epic-less stories). This container rule adds one level **above** epics only. Record the resolved container key in the run so a re-sync reuses it.

## Atlassian MCP tools

- Read: `getAccessibleAtlassianResources`, `getVisibleJiraProjects`, `getJiraProjectIssueTypesMetadata`, `getJiraIssue`, `lookupJiraAccountId`, `searchJiraIssuesUsingJql`, `getTransitionsForJiraIssue`.
- Write: `createJiraIssue`, `editJiraIssue`, `addCommentToJiraIssue`, `transitionJiraIssue`.

## Operational notes

- **JQL `labels` has no `LIKE`/wildcard.** To find existing trace-labelled issues, enumerate the exact labels in one `labels IN ("trace:EPIC-…","trace:STORY-…", …)` set (or one query per label). `labels LIKE "trace:*"` is rejected by JQL.
- **Large MCP responses overflow context.** `getVisibleJiraProjects` on a busy site can exceed the tool-result limit and be spilled to a file. Parse it from that file (jq/python) or narrow it with `searchString` — don't read the whole blob inline.
- **Re-push Descriptions from the file, not from memory.** When updating an existing issue's Description, read the card and push a verbatim slice (body minus H1, minus `## Jira Sync`, minus the `<!-- pipeline-only -->` block); reconstructing the body by hand risks transcription typos and a needless corrective round-trip.

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
- [ ] 5b. On each *updated* issue with a real field diff: one Hungarian change-summary comment posted (create path skipped; no-op re-sync posts nothing)
- [ ] 6. Jira-owned fields pulled into each card's ## Jira Sync block (diffed, not clobbered)
- [ ] 6b. (opt-in) Cross-reference backlinks written into card bodies; affected Descriptions re-pushed
- [ ] 6c. Dependency issue links created from each card's ## Dependency Edges (Blocks/Relates); idempotent + inverse-deduped; external/dangling skipped; drift reported
- [ ] 7. Every artifact reconciled 1:1 (no duplicate issues, no orphan stories); outcome rows reported
```

1. **Reach Jira, resolve context.** Confirm the Atlassian MCP tools exist (if not, stop — the server is not wired/authenticated). `getAccessibleAtlassianResources` → `cloudId`; resolve the project key (recorded block or AskUserQuestion); `getJiraProjectIssueTypesMetadata` → the project's Epic/Story issue-type ids. Completion: cloudId, project, and issue-type ids in hand.

2. **Enumerate artifacts.** List every `openspec/backlog/epics/EPIC-<epic-slug>.md` and `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md`, each with its trace ID from the card (pair a story directory to its epic by the epic filename minus its `EPIC-` prefix). If epics or stories are missing, stop and offer the producing step (`pte-openspec-to-epics` / `pte-openspec-to-stories` / `pte-openspec-bdd-tests`) rather than syncing a partial backlog. Completion: the full artifact set (or the flagged fallback) is named.

3. **Resolve + plan (dry-run).** For each artifact, find its issue by recorded key → trace label → (would-create). Print the plan as trace ID → Jira key (or "NEW") with the action per field owner. Completion: every artifact has a resolved action; the plan is printed and nothing is written yet.

4. **Confirm writes.** AskUserQuestion before the first write. Completion: the user has approved (or scoped down) the write set.

5. **Push local-owned fields.** Epics first: create/update each Epic (parent = the project-container issue when the project has a level above Epic — see *Epic parent*), push its **rollup** `Story points` (`ΣSP`) + `Original estimate` (`ΣEβ`h), and record its key; if any epic-less story is in the set, resolve/create the standalone collector epic (`trace:EPIC-standalone`) too. Then stories: create/update each Story with `parent` = its epic's Jira key (or the collector's key for epic-less stories), `trace:…` label, Summary from the H1 title (minus its ` · TRACE-ID` suffix), Description from the **Jira-visible card body** (minus the H1, the `## Jira Sync` block, and the `<!-- pipeline-only -->` block — see *Jira mapping*), the **Story points** field from the card's `## Estimation` SP, and the **Original estimate** from its `Becsült munkaóra` (`<Eβ>h`) — resolve the custom field id first; skip a structured push if the project lacks that field. Never touch Jira-owned fields (status/assignee/sprint). Completion: every artifact has a live issue; every story's parent is set; every epic + story has its SP + Original estimate pushed (or skipped-with-reason).

5b. **Comment the change on updates.** For each issue that was **updated** (not created) and whose locally-owned fields actually differed from Jira, post one Hungarian change-summary comment (`addCommentToJiraIssue`) listing the changed fields with old → new — per *Change-summary comment on update*. Skip newly-created issues and no-op re-syncs entirely. Completion: every genuinely-changed issue carries its audit comment; no comment on create or no-op.

6. **Pull Jira-owned fields.** For each issue, read status/assignee/sprint/key/URL and write the `## Jira Sync` block into its card — replace only that section, diff before overwriting. Completion: every card's block reflects the issue's current Jira-owned state.

6b. **Cross-reference backlinks (opt-in).** Only when the user asks for it: with every key now assigned, linkify the body cross-references per *Cross-reference backlinks* above (idempotent, exact format, skip the H1/gherkin-tags/Jira-Sync block), then re-push each affected Description. Completion: every card's body cross-reference carries its `([<KEY>](url))` link and Jira mirrors it.

6c. **Dependency issue links (default on).** With every key assigned, read each card's `## Dependency Edges` block and create the Jira links per *Dependency issue links* above: `blocks`/`depends-on` → the *Blocks* link type (correct direction), `relates-to` → *Relates*, `external` skipped. Resolve the link type localization-tolerantly; read existing `issuelinks` first so it is idempotent and inverse-deduped; skip dangling targets; report additive drift (never delete). Completion: every non-external, non-dangling edge has exactly one live Jira link, and any link with no matching edge is reported as drift.

7. **Verify exhaustively.** Every epic and story maps to exactly one issue (no duplicates from a missed match); every story's `parent` resolves to its epic's issue; every card carries a `## Jira Sync` block with a real key; every non-external, non-dangling `## Dependency Edges` edge has exactly one Jira link in the correct direction (no duplicate/reversed link), and any drift link is reported not deleted; no locally-owned field was pulled from Jira and no Jira-owned field was pushed from local. Report any artifact you could not reconcile cleanly rather than guessing.
