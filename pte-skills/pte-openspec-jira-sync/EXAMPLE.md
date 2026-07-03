# Worked example: reconcile one epic + its story to Jira

One end-to-end reconcile showing the non-obvious moves under the **new schema**: **epics before stories** (so the story's `parent` has a key), matching by **trace label** when the card has no recorded key yet, resolving the localized **"Eposz"** issue type, pushing only the **locally-owned** Description slice (H1 + `## Jira Sync` + the `<!-- pipeline-only -->` block all removed), and pulling the **Jira-owned** status back into a `## Jira Sync` block.

The key teaching point is the **Description slice**. The epic file on disk ends with a pipeline-only fence carrying its `## User Stories` list and `## Source Spec Reference` map. That region is pipeline-internal — the downstream skills read it, Jira never sees it. So when this skill pushes the epic's Description it slices the file from disk and drops **three** things: the **H1** (that becomes the Summary), the **`## Jira Sync`** block (a local record of Jira-owned state — pushing it back would be circular), and the **whole pipeline-only region** (markers inclusive). What lands in Jira is only the English-headed prose sections. The on-disk epic and the resulting Jira Description are shown side by side below.

Project resolved earlier in the run: `cloudId = 11111111-2222-3333-4444-555555555555`, project key `INYO`, Epic/Story issue types confirmed via `getJiraProjectIssueTypesMetadata` — this is a **Hungarian** project, so the Epic type's display name is **"Eposz"** (resolved by `untranslatedName: "Epic"` / `hierarchyLevel: 1`); Story stays "Story".

## Input — the artifacts

`openspec/backlog/epics/EPIC-belso-hozzaferes.md` (trace `EPIC-belso-hozzaferes`) and one story under it, `openspec/backlog/stories/belso-hozzaferes/STORY-belso-hozzaferes-m365-szerepkorok.md` (trace `STORY-belso-hozzaferes-m365-szerepkorok`).

**The epic on disk** — outline, with the pipeline-only block present at the end (bodies elided with `…`; the point is the section structure and the slicing boundaries, not the full prose):

```markdown
# Belső hozzáférés — M365 auth, táblázat, audit · EPIC-belso-hozzaferes

## Epic
A belső kör (admin és HR/vezetők) M365-tel (Entra ID / OIDC) lép be. …

## Persona
- Admin (Laci) — `admin` szerepkör, szerkeszt. …

## E2E Scenario
Ha a belső felhasználók M365-tel, szerepkör szerint lépnek be, … mérve az
auditált műveletek lefedettségével …

## Problem / Solution
**Probléma:** A mai folyamat nem visszakövethető … **Megoldás:** Egy M365-alapú, szerepkörös belépés …

## Cross-cutting Concerns
- Biztonság: szerepkör-szeparáció szerver oldalon kikényszerítve. …

## MVP and Out of Scope
**MVP:** … **Out of Scope:** …

## Success metrics
- Reader-ből indított szerkesztő műveletek 100%-ban megtagadva. …

## Risks and Dependencies
- Függőség: Entra ID tenant + szerepkör-hozzárendelés. …

## High Level Acceptance Criteria
- M365-tel belépő admin eléri a feltöltés / override / lezárás műveleteket. …

<!-- pipeline-only:start -->
## Estimation
- **Becsült munkaóra:** 35 ó (≈ 5,8 ideális nap)
- **Story Point:** 18

## Dependency Edges
- external: Entra ID tenant

## User Stories
- `STORY-belso-hozzaferes-m365-szerepkorok`
  Mint belső felhasználó, szeretném M365-tel belépni … csak az admin
  szerkeszthessen, a reader pedig biztonságosan csak olvasson.
- `STORY-belso-hozzaferes-tablazat-nezet` …
- `STORY-belso-hozzaferes-audit-naplo` …

## Source Spec Reference

| Story ID | Epic ID | Source `### Requirement` | Covered `#### Scenario`s |
|----------|---------|--------------------------|----------------------------|
| STORY-belso-hozzaferes-m365-szerepkorok | EPIC-belso-hozzaferes | M365 alapú belső hozzáférés és szerepkörök | Admin szerkeszthet; Reader csak olvas |
| STORY-belso-hozzaferes-tablazat-nezet | EPIC-belso-hozzaferes | Szűrhető, aggregálható táblázat-nézet | Szűrés átcsúszott tételekre; Nyers adóazonosító elrejtése |
| STORY-belso-hozzaferes-audit-naplo | EPIC-belso-hozzaferes | Teljes audit napló | Státuszváltás naplózása; Token-elfogadás jelölése |
<!-- pipeline-only:end -->
```

**The story on disk** — the lean card (bodies elided):

```markdown
# M365 belépés és szerepkörök · STORY-belso-hozzaferes-m365-szerepkorok

## User Story
Mint **belső felhasználó**, szeretném **M365-tel belépni és a szerepkörömnek
megfelelő jogot kapni**, hogy **csak az admin szerkeszthessen, a reader pedig
biztonságosan csak olvasson**. …

## Context
A szerepkör-szeparáció védi a szerkesztő műveleteket (feltöltés, override, lezárás). …

## BDD Test
```gherkin
# language: hu
@EPIC-belso-hozzaferes @STORY-belso-hozzaferes-m365-szerepkorok
Jellemző: belso-hozzaferes
  Szabály: M365 alapú belső hozzáférés és szerepkörök
    …
```

## Risks and Dependencies
- Függőség: Entra ID (M365/OIDC) tenant és szerepkör-hozzárendelés. …

<!-- pipeline-only:start -->
Parent epic: `EPIC-belso-hozzaferes`

## Estimation
- **3-Points becslés (ideális óra):** O 6 / M 8 / P 16 | **Eβ 9**, σ 1,7
- **Becsült munkaóra:** 9 ó (≈ 1,5 ideális nap)
- **Story Point:** 5

## Dependency Edges
- blocks: `STORY-belso-hozzaferes-tablazat-nezet`
- blocks: `STORY-belso-hozzaferes-audit-naplo`
- external: Entra ID (M365/OIDC) tenant
<!-- pipeline-only:end -->
```

Neither artifact has a `## Jira Sync` block yet — this is the first sync. All three of the epic's stories are in the sync set; after step 5 they hold keys `INYO-23` (m365-szerepkorok), `INYO-24` (tablazat-nezet), `INYO-25` (audit-naplo).

## Resolve + dry-run plan

- Epic: no recorded key → `searchJiraIssuesUsingJql` `labels = "trace:EPIC-belso-hozzaferes"` → **hit `INYO-18`** → **update Eposz**.
- Story: no recorded key → `labels = "trace:STORY-belso-hozzaferes-m365-szerepkorok"` → **hit `INYO-23`** → **update Story**, parent = the epic's key `INYO-18`.

Issue types resolved against the project metadata by `untranslatedName` / `hierarchyLevel` — **not** by display name, because a Hungarian project localizes them: the Epic type displays as **"Eposz"** (`untranslatedName: "Epic"`, `hierarchyLevel: 1`), Story as "Story" (`hierarchyLevel: 0`). The resolved **display** name ("Eposz" / "Story") is what goes into `issueTypeName` on any create.

```
Terv (dry-run) — cloudId …555555, projekt INYO:
  trace ID                                 Jira key     action       type (untranslatedName)
  EPIC-belso-hozzaferes                    INYO-18      update       Eposz  (Epic / hierarchy 1)
  STORY-belso-hozzaferes-m365-szerepkorok  INYO-23      update       Story  (Story / hierarchy 0)
                                           parent → INYO-18
Ír-e? (editJiraIssue ×2 — Summary + Description + becslés-mezők (Story points, Original estimate); státusz nem íródik)
```

User confirms.

## Push — the Description slice (H1 + pipeline-only + Jira Sync removed)

Epic first. Summary = the H1 minus its ` · EPIC-…` suffix → `Belső hozzáférés — M365 auth, táblázat, audit`; label `trace:EPIC-belso-hozzaferes`; the **rollup structured fields** pushed from the fenced `## Estimation` — **Story points = 18** (`ΣSP`) and **Original estimate = 35h** (`ΣEβ`). The **Description** is the file sliced from disk with the three excluded regions dropped — so Jira receives only the English-headed prose, **not** the `## Estimation` rollup, the `## User Stories` list or the `## Source Spec Reference` map:

```markdown
## Epic
A belső kör (admin és HR/vezetők) M365-tel (Entra ID / OIDC) lép be. …

## Persona
- Admin (Laci) — `admin` szerepkör, szerkeszt. …

## E2E Scenario
Ha a belső felhasználók M365-tel, szerepkör szerint lépnek be, … mérve az
auditált műveletek lefedettségével …

## Problem / Solution
**Probléma:** A mai folyamat nem visszakövethető … **Megoldás:** Egy M365-alapú, szerepkörös belépés …

## Cross-cutting Concerns
- Biztonság: szerepkör-szeparáció szerver oldalon kikényszerítve. …

## MVP and Out of Scope
**MVP:** … **Out of Scope:** …

## Success metrics
- Reader-ből indított szerkesztő műveletek 100%-ban megtagadva. …

## Risks and Dependencies
- Függőség: Entra ID tenant + szerepkör-hozzárendelés. …

## High Level Acceptance Criteria
- M365-tel belépő admin eléri a feltöltés / override / lezárás műveleteket. …
```

The `# Belső hozzáférés …` H1 became the Summary; the entire `<!-- pipeline-only:start -->…<!-- pipeline-only:end -->` region (the `## Estimation` rollup, the `## Dependency Edges` block, the `## User Stories` list + the `## Source Spec Reference` map) is gone; and there is no `## Jira Sync` block yet to drop — but on the re-run there will be, and it too stays out of the Description. Slice from the file, verbatim — do not reconstruct from memory.

Then the story, with `parent = INYO-18`. Summary = `M365 belépés és szerepkörök`; label `trace:STORY-belso-hozzaferes-m365-szerepkorok`; **Story points = 5** and **Original estimate = 9h**, pushed from `## Estimation` into the structured fields. The story's `## Estimation` sits in its **pipeline-only fence**, so the Description slice drops it along with the H1 and (on re-run) the `## Jira Sync` block — leaving `## User Story` + `## Context` + `## BDD Test` (Gherkin and all) + `## Risks and Dependencies`:

```markdown
## User Story
Mint **belső felhasználó**, szeretném **M365-tel belépni és a szerepkörömnek
megfelelő jogot kapni**, hogy **csak az admin szerkeszthessen, a reader pedig
biztonságosan csak olvasson**. …

## Context
A szerepkör-szeparáció védi a szerkesztő műveleteket (feltöltés, override, lezárás). …

## BDD Test
```gherkin
# language: hu
@EPIC-belso-hozzaferes @STORY-belso-hozzaferes-m365-szerepkorok
Jellemző: belso-hozzaferes
  Szabály: M365 alapú belső hozzáférés és szerepkörök
    …
```

## Risks and Dependencies
- Függőség: Entra ID (M365/OIDC) tenant és szerepkör-hozzárendelés. …
```

Note that the H1, the `Parent epic:` line, and the `## Estimation` section are **all dropped** from the Description: the H1 is the Summary; the `Parent epic:` line and `## Estimation` live **inside the pipeline-only fence**, so the slice removes them. The epic↔story link is the Jira `parent` field (`INYO-18`), and the estimate is pushed to **structured** fields — **SP (5)** → *Story points*, **Original estimate (9h)** → time-tracking — both local-owned (resolve the custom field id first). No status was set or transitioned — status is Jira-owned.

## Pull — Jira-owned state back onto disk

`getJiraIssue INYO-18` / `INYO-23` → read status, key, URL. Write (or replace) the `## Jira Sync` block in each artifact — English heading, keys/URLs verbatim, only this section touched. Into the epic file:

```markdown
## Jira Sync
- Jira key: INYO-18
- Issue type: Eposz
- Status: Nyitás
- URL: https://pte-politechnika.atlassian.net/browse/INYO-18
- Last sync: 2026-07-01T18:00:00+02:00
```

And the analogous block into the story card:

```markdown
## Jira Sync
- Jira key: INYO-23
- Issue type: Story
- Status: Nyitás
- URL: https://pte-politechnika.atlassian.net/browse/INYO-23
- Last sync: 2026-07-01T18:00:00+02:00
```

`Státusz: Nyitás` is the Jira board's localized state pulled straight from the issue — this skill records it, it never sets it. On the next push, this block sits between the last visible section and (in the epic) the pipeline-only fence, and it is one of the three regions the Description slice drops.

```
Kész (outcome) — projekt INYO:
  EPIC-belso-hozzaferes                    INYO-18   updated + pulled (Nyitás)
  STORY-belso-hozzaferes-m365-szerepkorok  INYO-23   updated + pulled (Nyitás), parent INYO-18
```

## Dependency links — the 6c step (default on)

With all three stories now keyed, the sync mirrors the `## Dependency Edges` blocks as real Jira issue links. The `m365-szerepkorok` card declares `blocks` on its two siblings; its `external:` Entra ID line has no trace ID, so it is skipped (a Jira link needs two issues).

Resolve the link type first — this Hungarian site localizes it, so `getIssueLinkTypes` returns the *Blocks* family displayed as **"Blokkolja / Blokkolva"**; match it by its stable name, not the display string. Then, reading `INYO-23`'s existing `issuelinks` to stay idempotent, create the two links in the correct direction:

```
Függőségi linkek (6c) — forrás: STORY-…-m365-szerepkorok ## Dependency Edges:
  blocks  STORY-…-tablazat-nezet  →  INYO-23 "Blokkolja" INYO-24   (createIssueLink)
  blocks  STORY-…-audit-naplo     →  INYO-23 "Blokkolja" INYO-25   (createIssueLink)
  external "Entra ID tenant"       →  kihagyva (nincs Jira-issue)
Kész: 2 létrehozva, 0 már megvolt, 1 kihagyva (external), 0 drift.
```

Direction is load-bearing: `blocks: STORY-tablazat-nezet` on the m365 card becomes `INYO-23` **blocks** `INYO-24` (outward), so the board reads it the same way the edge does. Had the `tablazat-nezet` card also carried `depends-on: STORY-…-m365-szerepkorok`, that is the **same** link stated from the other end — the inverse-dedup drops it, so `INYO-24` gets no second, reversed link. On a re-run both links already exist, so 6c creates nothing. If later someone deletes the `blocks: STORY-…-audit-naplo` edge on disk, the `INYO-23 → INYO-25` link is **reported as drift**, never auto-deleted — a human removes it if they mean to.

## Re-run — the change-summary comment on update

The first sync above **created** the issues, so no summary comment was posted (the issue is its own record). Now say a later edit bumps the story's estimate — `## Estimation` goes from `Story Point: 5` / `Becsült munkaóra: 9 ó` to `8` / `18 ó` — and the run resolves `INYO-23` by its recorded `Jira key`. Because this is an **update** and the pushed fields differ from Jira, the skill posts **one** comment on `INYO-23` (`addCommentToJiraIssue`) after the field push lands:

```markdown
🔄 Pipeline szinkron — 2026-07-02T09:15:00+02:00
Változott mezők:
- Story points: 5 → 8
- Original estimate: 9h → 18h
```

If instead nothing had changed — every locally-owned field identical to Jira — the re-run posts **no** comment and the outcome row reads `no-op`; the audit trail only grows when something actually moved. A `created` issue never gets one either. Only the Description text-changed case collapses to a bare `Leírás frissítve` line (no old → new text dump).

## Why these moves

- **One owner per field** — Summary, Description, `trace:…` label, issue type, epic↔story `parent`, and **Story points** (from `## Estimation`) are **local**-owned (pushed, overwrite); status/assignee/sprint/key/URL are **Jira**-owned (pulled into `## Jira Sync`). A field never crosses against its owner, so a re-run never clobbers either side. Story points is a deliberate flip from the old Jira-owned estimate — the AI `## Estimation` is the reference base.
- **The pipeline-only block never reaches Jira** — the Description slice drops the whole `<!-- pipeline-only:start -->…<!-- pipeline-only:end -->` region, so the epic's `## User Stories` list and `## Source Spec Reference` map stay on disk for `pte-openspec-to-stories` / `pte-openspec-bdd-tests` while Jira sees only the English-headed prose. Three regions are excluded from every Description push: the **H1** (→ Summary), the **`## Jira Sync`** block (local record of Jira-owned state — circular to push), and the **pipeline-only** region.
- **Slice from disk, don't reconstruct** — the pushed Description is a verbatim file slice cut by those three boundaries, not sections retyped from memory; that avoids transcription drift on re-push.
- **Status is Jira-owned — pulled, never pushed** — `Státusz: Nyitás` came from `getJiraIssue`; no `transitionJiraIssue` is called unless the user explicitly asks. The board owns the workflow state.
- **Issue type by untranslatedName, not display name** — a Hungarian project names the Epic type **"Eposz"**; it is resolved by `untranslatedName: "Epic"` / `hierarchyLevel: 1` and the resolved display name is passed to any create. Story stays "Story" (`hierarchyLevel: 0`).
- **Updates leave an audit comment; creates and no-ops don't** — when a re-sync changes a locally-owned field on an existing issue, one Hungarian change-summary comment (`🔄 Pipeline szinkron — …`, changed fields with old → new) lands on the issue so a team watching the board sees why it moved. It is gated on a real field diff, not on the update path: a no-op re-sync stays silent, and a freshly created issue gets none (it is its own record). Write-only — the skill never reads comments back, so "comments are Jira-owned" still holds.
- **Matching is idempotent via the trace label** — with no recorded `Jira key` yet, each artifact re-bound to its existing issue (`INYO-18` / `INYO-23`) through its `trace:…` label instead of creating a duplicate. Once the `## Jira Sync` block records the key, the next run matches on that first — the reconcile is a no-op across machines and fresh clones.
- **Dependency links are local-owned, additive, direction-exact** — the `## Dependency Edges` block is the source (never the `## Risks and Dependencies` prose, never `DEPENDENCY_GRAPH.md`); `blocks`/`depends-on` become *Blocks* links resolved localization-tolerantly ("Blokkolja"), `relates-to` becomes *Relates*, `external` is skipped. Reading existing `issuelinks` first makes 6c idempotent and inverse-deduped; a link with no matching edge is reported as **drift**, never deleted — the board may carry links the backlog doesn't own.
