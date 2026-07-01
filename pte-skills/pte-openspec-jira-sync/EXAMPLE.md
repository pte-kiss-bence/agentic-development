# Worked example: reconcile one epic + its story to Jira

One end-to-end reconcile showing the non-obvious moves under the **new schema**: **epics before stories** (so the story's `parent` has a key), matching by **trace label** when the card has no recorded key yet, resolving the localized **"Eposz"** issue type, pushing only the **locally-owned** Description slice (H1 + `## Jira szinkron` + the `<!-- pipeline-only -->` block all removed), and pulling the **Jira-owned** status back into a `## Jira szinkron` block.

The key teaching point is the **Description slice**. The epic file on disk ends with a pipeline-only fence carrying its `## User story-k` list and `## Forrás-spec hivatkozás` map. That region is pipeline-internal — the downstream skills read it, Jira never sees it. So when this skill pushes the epic's Description it slices the file from disk and drops **three** things: the **H1** (that becomes the Summary), the **`## Jira szinkron`** block (a local record of Jira-owned state — pushing it back would be circular), and the **whole pipeline-only region** (markers inclusive). What lands in Jira is only the English-headed prose sections. The on-disk epic and the resulting Jira Description are shown side by side below.

Project resolved earlier in the run: `cloudId = 11111111-2222-3333-4444-555555555555`, project key `INYO`, Epic/Story issue types confirmed via `getJiraProjectIssueTypesMetadata` — this is a **Hungarian** project, so the Epic type's display name is **"Eposz"** (resolved by `untranslatedName: "Epic"` / `hierarchyLevel: 1`); Story stays "Story".

## Input — the artifacts

`openspec/backlog/epics/EPIC-belso-hozzaferes.md` (trace `EPIC-belso-hozzaferes`) and one story under it, `openspec/backlog/stories/belso-hozzaferes/STORY-belso-hozzaferes-m365-szerepkorok.md` (trace `STORY-belso-hozzaferes-m365-szerepkorok`).

**The epic on disk** — outline, with the pipeline-only block present at the end (bodies elided with `…`; the point is the section structure and the slicing boundaries, not the full prose):

```markdown
# Belső hozzáférés — M365 auth, táblázat, audit · EPIC-belso-hozzaferes

## Description
A belső kör (admin és HR/vezetők) M365-tel (Entra ID / OIDC) lép be. …

## Persona
- Admin (Laci) — `admin` szerepkör, szerkeszt. …

## E2E Scenario
Ha a belső felhasználók M365-tel, szerepkör szerint lépnek be, … mérve az
auditált műveletek lefedettségével …

## Problem / Solution
P: A mai folyamat nem visszakövethető … S: Egy M365-alapú, szerepkörös belépés …

## Cross-cutting Concerns
- Biztonság: szerepkör-szeparáció szerver oldalon kikényszerítve. …

## MVP and Out of Scope
MVP: … Out of Scope: …

## Success metrics
- Reader-ből indított szerkesztő műveletek 100%-ban megtagadva. …

## Risks and Dependencies
- Függőség: Entra ID tenant + szerepkör-hozzárendelés. …

## High Level Acceptance Criteria
- M365-tel belépő admin eléri a feltöltés / override / lezárás műveleteket. …

<!-- pipeline-only:start -->
## User story-k
- `STORY-belso-hozzaferes-m365-szerepkorok`
  Mint belső felhasználó, szeretném M365-tel belépni … csak az admin
  szerkeszthessen, a reader pedig biztonságosan csak olvasson.
- `STORY-belso-hozzaferes-tablazat-nezet` …
- `STORY-belso-hozzaferes-audit-naplo` …

## Forrás-spec hivatkozás

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| STORY-belso-hozzaferes-m365-szerepkorok | EPIC-belso-hozzaferes | M365 alapú belső hozzáférés és szerepkörök | Admin szerkeszthet; Reader csak olvas |
| STORY-belso-hozzaferes-tablazat-nezet | EPIC-belso-hozzaferes | Szűrhető, aggregálható táblázat-nézet | Szűrés átcsúszott tételekre; Nyers adóazonosító elrejtése |
| STORY-belso-hozzaferes-audit-naplo | EPIC-belso-hozzaferes | Teljes audit napló | Státuszváltás naplózása; Token-elfogadás jelölése |
<!-- pipeline-only:end -->
```

**The story on disk** — the lean card (bodies elided):

```markdown
# M365 belépés és szerepkörök · STORY-belso-hozzaferes-m365-szerepkorok

Parent epic: `EPIC-belso-hozzaferes`

## Description
Mint **belső felhasználó**, szeretném **M365-tel belépni és a szerepkörömnek
megfelelő jogot kapni**, hogy **csak az admin szerkeszthessen, a reader pedig
biztonságosan csak olvasson**. …

## Context
A szerepkör-szeparáció védi a szerkesztő műveleteket (feltöltés, override, lezárás). …

## BDD Test
```gherkin
# language: hu
@EPIC-belso-hozzaferes @STORY-belso-hozzaferes-m365-szerepkorok
Jellemző: M365 alapú belső hozzáférés és szerepkörök
  …
```

## Risks and Dependencies
- Függőség: Entra ID (M365/OIDC) tenant és szerepkör-hozzárendelés. …
```

Neither artifact has a `## Jira szinkron` block yet — this is the first sync.

## Resolve + dry-run plan

- Epic: no recorded key → `searchJiraIssuesUsingJql` `labels = "trace:EPIC-belso-hozzaferes"` → **hit `INYO-18`** → **update Eposz**.
- Story: no recorded key → `labels = "trace:STORY-belso-hozzaferes-m365-szerepkorok"` → **hit `INYO-23`** → **update Story**, parent = the epic's key `INYO-18`.

Issue types resolved against the project metadata by `untranslatedName` / `hierarchyLevel` — **not** by display name, because a Hungarian project localizes them: the Epic type displays as **"Eposz"** (`untranslatedName: "Epic"`, `hierarchyLevel: 1`), Story as "Story" (`hierarchyLevel: 0`). The resolved **display** name ("Eposz" / "Story") is what goes into `issueTypeName` on any create.

```
Terv (dry-run) — cloudId …555555, projekt INYO:
  trace ID                                 Jira kulcs   akció        típus (untranslatedName)
  EPIC-belso-hozzaferes                    INYO-18      update       Eposz  (Epic / hierarchy 1)
  STORY-belso-hozzaferes-m365-szerepkorok  INYO-23      update       Story  (Story / hierarchy 0)
                                           parent → INYO-18
Ír-e? (editJiraIssue ×2 — Summary + Description; státusz nem íródik)
```

User confirms.

## Push — the Description slice (H1 + pipeline-only + Jira szinkron removed)

Epic first. Summary = the H1 minus its ` · EPIC-…` suffix → `Belső hozzáférés — M365 auth, táblázat, audit`; label `trace:EPIC-belso-hozzaferes`. The **Description** is the file sliced from disk with the three excluded regions dropped — so Jira receives only the English-headed prose, **not** the `## User story-k` list or the `## Forrás-spec hivatkozás` map:

```markdown
## Description
A belső kör (admin és HR/vezetők) M365-tel (Entra ID / OIDC) lép be. …

## Persona
- Admin (Laci) — `admin` szerepkör, szerkeszt. …

## E2E Scenario
Ha a belső felhasználók M365-tel, szerepkör szerint lépnek be, … mérve az
auditált műveletek lefedettségével …

## Problem / Solution
P: A mai folyamat nem visszakövethető … S: Egy M365-alapú, szerepkörös belépés …

## Cross-cutting Concerns
- Biztonság: szerepkör-szeparáció szerver oldalon kikényszerítve. …

## MVP and Out of Scope
MVP: … Out of Scope: …

## Success metrics
- Reader-ből indított szerkesztő műveletek 100%-ban megtagadva. …

## Risks and Dependencies
- Függőség: Entra ID tenant + szerepkör-hozzárendelés. …

## High Level Acceptance Criteria
- M365-tel belépő admin eléri a feltöltés / override / lezárás műveleteket. …
```

The `# Belső hozzáférés …` H1 became the Summary; the entire `<!-- pipeline-only:start -->…<!-- pipeline-only:end -->` region (the `## User story-k` list + the `## Forrás-spec hivatkozás` map) is gone; and there is no `## Jira szinkron` block yet to drop — but on the re-run there will be, and it too stays out of the Description. Slice from the file, verbatim — do not reconstruct from memory.

Then the story, with `parent = INYO-18`. Summary = `M365 belépés és szerepkörök`; label `trace:STORY-belso-hozzaferes-m365-szerepkorok`; **Story points = 5**, pushed from `## Estimation` into the structured field. The story carries **no** pipeline-only fence (its map lives in the epic), so its Description slice is the whole visible card minus the H1 and (on re-run) the `## Jira szinkron` block — `## Description` + `## Context` + `## BDD Test` (Gherkin and all) + `## Estimation` + `## Risks and Dependencies`:

```markdown
## Description
Mint **belső felhasználó**, szeretném **M365-tel belépni és a szerepkörömnek
megfelelő jogot kapni**, hogy **csak az admin szerkeszthessen, a reader pedig
biztonságosan csak olvasson**. …

## Context
A szerepkör-szeparáció védi a szerkesztő műveleteket (feltöltés, override, lezárás). …

## BDD Test
```gherkin
# language: hu
@EPIC-belso-hozzaferes @STORY-belso-hozzaferes-m365-szerepkorok
Jellemző: M365 alapú belső hozzáférés és szerepkörök
  …
```

## Estimation
- **PERT (ideális fejlesztői óra):** O 4 / M 8 / P 16 → **Eβ ≈ 9 ó**, σ ≈ 2 ó
- **Story point:** 5

## Risks and Dependencies
- Függőség: Entra ID (M365/OIDC) tenant és szerepkör-hozzárendelés. …
```

Note the `Parent epic:` metadata line and the H1 are both dropped: the H1 is the Summary, and the epic↔story link is the Jira `parent` field (`INYO-18`), not body prose. The `## Estimation` section rides along **inside** the Description, and its **SP (5)** is *additionally* pushed to the structured **Story points** field (local-owned — resolve the custom field id first). No status was set or transitioned — status is Jira-owned.

## Pull — Jira-owned state back onto disk

`getJiraIssue INYO-18` / `INYO-23` → read status, key, URL. Write (or replace) the `## Jira szinkron` block in each artifact — Hungarian heading kept, keys/URLs verbatim, only this section touched. Into the epic file:

```markdown
## Jira szinkron
- Jira kulcs: INYO-18
- Issue típus: Eposz
- Státusz: Nyitás
- URL: https://pte-politechnika.atlassian.net/browse/INYO-18
- Utolsó szinkron: 2026-07-01T18:00:00+02:00
```

And the analogous block into the story card:

```markdown
## Jira szinkron
- Jira kulcs: INYO-23
- Issue típus: Story
- Státusz: Nyitás
- URL: https://pte-politechnika.atlassian.net/browse/INYO-23
- Utolsó szinkron: 2026-07-01T18:00:00+02:00
```

`Státusz: Nyitás` is the Jira board's localized state pulled straight from the issue — this skill records it, it never sets it. On the next push, this block sits between the last visible section and (in the epic) the pipeline-only fence, and it is one of the three regions the Description slice drops.

```
Kész (outcome) — projekt INYO:
  EPIC-belso-hozzaferes                    INYO-18   updated + pulled (Nyitás)
  STORY-belso-hozzaferes-m365-szerepkorok  INYO-23   updated + pulled (Nyitás), parent INYO-18
```

## Why these moves

- **One owner per field** — Summary, Description, `trace:…` label, issue type, epic↔story `parent`, and **Story points** (from `## Estimation`) are **local**-owned (pushed, overwrite); status/assignee/sprint/key/URL are **Jira**-owned (pulled into `## Jira szinkron`). A field never crosses against its owner, so a re-run never clobbers either side. Story points is a deliberate flip from the old Jira-owned estimate — the AI `## Estimation` is the reference base.
- **The pipeline-only block never reaches Jira** — the Description slice drops the whole `<!-- pipeline-only:start -->…<!-- pipeline-only:end -->` region, so the epic's `## User story-k` list and `## Forrás-spec hivatkozás` map stay on disk for `pte-openspec-to-stories` / `pte-openspec-bdd-tests` while Jira sees only the English-headed prose. Three regions are excluded from every Description push: the **H1** (→ Summary), the **`## Jira szinkron`** block (local record of Jira-owned state — circular to push), and the **pipeline-only** region.
- **Slice from disk, don't reconstruct** — the pushed Description is a verbatim file slice cut by those three boundaries, not sections retyped from memory; that avoids transcription drift on re-push.
- **Status is Jira-owned — pulled, never pushed** — `Státusz: Nyitás` came from `getJiraIssue`; no `transitionJiraIssue` is called unless the user explicitly asks. The board owns the workflow state.
- **Issue type by untranslatedName, not display name** — a Hungarian project names the Epic type **"Eposz"**; it is resolved by `untranslatedName: "Epic"` / `hierarchyLevel: 1` and the resolved display name is passed to any create. Story stays "Story" (`hierarchyLevel: 0`).
- **Matching is idempotent via the trace label** — with no recorded `Jira kulcs` yet, each artifact re-bound to its existing issue (`INYO-18` / `INYO-23`) through its `trace:…` label instead of creating a duplicate. Once the `## Jira szinkron` block records the key, the next run matches on that first — the reconcile is a no-op across machines and fresh clones.
