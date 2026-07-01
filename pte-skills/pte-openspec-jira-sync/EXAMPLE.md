# Worked example: reconcile one epic + its story to Jira

One end-to-end reconcile showing the non-obvious moves: **epics before stories** (so the story's `parent` has a key), matching by **trace label** when the card has no recorded key yet, pushing only **locally-owned** fields, and pulling the **Jira-owned** status back into a `## Jira szinkron` block.

Project resolved earlier in the run: `cloudId = 11111111-2222-3333-4444-555555555555`, project key `PTE`, Epic/Story issue types confirmed via `getJiraProjectIssueTypesMetadata`.

## Input — the artifacts

`openspec/backlog/epics/elofizetes-cikkhozzaferes.md` (trace `EPIC-elofizetes-cikkhozzaferes`) and one story under it:

`openspec/backlog/stories/elofizetes-cikkhozzaferes/elofizetesi-szint-szerinti-cikkhozzaferes.md`

```markdown
# Cím
Szintnek megfelelő cikkhozzáférés —
`STORY-elofizetes-cikkhozzaferes-elofizetesi-szint-szerinti-cikkhozzaferes`
(`EPIC-elofizetes-cikkhozzaferes`)

## User story
Mint olvasó, szeretném a szintemnek megfelelő cikkeket látni, hogy csak a
jogosult tartalomhoz férjek hozzá.

## Kontextus
Az olvasó előfizetési szintje dönti el, mely cikkeket látja.
...
```

Neither artifact has a `## Jira szinkron` block yet — this is the first sync.

## Resolve + dry-run plan

- Epic: no recorded key → `searchJiraIssuesUsingJql` `labels = "trace:EPIC-elofizetes-cikkhozzaferes"` → no hit → **NEW Epic**.
- Story: no recorded key → `labels = "trace:STORY-…-elofizetesi-szint-szerinti-cikkhozzaferes"` → no hit → **NEW Story**, parent = the epic (key known only after the epic is created).

```
Terv (dry-run) — cloudId PTE:
  EPIC-elofizetes-cikkhozzaferes   → NEW Epic
  STORY-…-elofizetesi-szint-…      → NEW Story (parent: EPIC-… fenti)
Ír-e? (createJiraIssue ×2)
```

User confirms.

## Push — epics first, then stories

1. `createJiraIssue` Epic → returns `PTE-100`. Summary = the epic's **Cím**; label `trace:EPIC-elofizetes-cikkhozzaferes`.
2. `createJiraIssue` Story with `parent = PTE-100` → returns `PTE-101`. Summary = the card's **Cím**; Description = **User story** + **Kontextus** + **Elfogadási kritériumok** + embedded **BDD teszt**; label `trace:STORY-…-elofizetesi-szint-szerinti-cikkhozzaferes`.

Only locally-owned fields were written. No status was set — the Story lands in the project's default status, which Jira owns.

## Pull — Jira-owned state back onto disk

`getJiraIssue PTE-101` → status `To Do`, no assignee yet. Append the block to the story card (and the parallel block, with `PTE-100`, to the epic file):

```markdown
## Jira szinkron
- Jira kulcs: PTE-101
- Issue típus: Story
- Státusz: To Do
- URL: https://pte.atlassian.net/browse/PTE-101
- Utolsó szinkron: 2026-07-01T10:12:00Z
```

## The re-run (idempotency)

A week later the team moved `PTE-101` to `In Progress` and the card's **Kontextus** was edited locally. Re-running:

- Match is instant via the recorded `Jira kulcs: PTE-101` — no duplicate created.
- Push: the edited **Kontextus** updates the Description (`editJiraIssue`) — local owns it.
- Pull: `Státusz` in the block updates `To Do → In Progress` — Jira owns it.
- The local edit and the board move both survive, because they touch different owners. That is the whole point of the ownership model.

## Variant — an epic-less story in the set

Say the sync set also contains `openspec/backlog/stories/hirlevel/egykattintasos-leiratkozas.md` (trace `STORY-hirlevel-egykattintasos-leiratkozas`), a card whose parent `EPIC-…` line is **empty** (authored by `pte-openspec-to-stories` in epic-less mode). It has no epic file to parent it, so:

- **Resolve the collector once:** `searchJiraIssuesUsingJql` `labels = "trace:EPIC-standalone"` → no hit → create one Epic (Summary *Önálló változtatások*, label `trace:EPIC-standalone`) → returns e.g. `PTE-90`. On later runs the label re-binds to `PTE-90` — never a second collector.
- **Parent the story to it:** `createJiraIssue` Story with `parent = PTE-90`, everything else (Summary, Description, `trace:STORY-…` label, pulled `## Jira szinkron` block) exactly as for an epic-backed story.

```
Terv (dry-run) — cloudId PTE:
  EPIC-standalone                       → resolve/NEW (gyűjtő-epic)
  STORY-hirlevel-egykattintasos-…       → NEW Story (parent: EPIC-standalone)
```

Only the parent resolution differs; every ownership and idempotency rule is unchanged.
