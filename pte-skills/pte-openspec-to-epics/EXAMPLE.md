# Worked example: OpenSpec spec → Agile Epic

One end-to-end transformation showing the non-obvious moves under the **new epic schema**: grouping a capability's Requirements into a single lean epic, writing the measurable hypothesis into `## E2E Scenario`, filling `## Cross-cutting Concerns` with real spec-derived concerns (not the rubric), assigning requirement-anchored trace IDs, and parking the *User Stories* list plus the traceability map inside the pipeline-only fence so `pte-openspec-jira-sync` hides them from Jira while the downstream skills still read them.

The pedagogical shape is unchanged: a short `## Input` spec excerpt, the `## Output` epic file, then a `## Why these moves` notes section. Section **headings inside the epic are English** (new schema); the **body stays Hungarian** (per `CONVENTIONS.md`); the pipeline-only headings and every `### Requirement` / `#### Scenario` title stay **verbatim**.

## Input — OpenSpec spec

`openspec/changes/aok-osszesito-mvp/specs/belso-hozzaferes/spec.md`

```markdown
### Requirement: M365 alapú belső hozzáférés és szerepkörök
A rendszer SHALL a belső kört (admin, HR/vezetők) M365-tel (Entra ID / OIDC) hitelesítse, és `admin` illetve `reader` szerepkör szerint engedélyezze a műveleteket.

#### Scenario: Admin szerkeszthet
- **WHEN** `admin` szerepkörű felhasználó M365-tel belép
- **THEN** a rendszer engedélyezi a feltöltés, override és lezárás műveleteket

#### Scenario: Reader csak olvas
- **WHEN** `reader` szerepkörű felhasználó szerkesztő műveletet kísérel meg
- **THEN** a rendszer szerver oldalon megtagadja a műveletet
- **THEN** a rendszer nem módosítja az adatot

### Requirement: Szűrhető, aggregálható táblázat-nézet
A rendszer SHALL az időszak adatait szűrhető, aggregálható táblázatként jelenítse meg, nyers adóazonosító nélkül.

#### Scenario: Szűrés átcsúszott tételekre
- **WHEN** a belső felhasználó az átcsúszott tételekre szűr
- **THEN** a rendszer csak a szűrésnek megfelelő sorokat mutatja
- **THEN** a rendszer megjeleníti az aggregált összesítést

#### Scenario: Nyers adóazonosító elrejtése
- **WHEN** a belső felhasználó a táblázat-nézetet vagy annak exportját megnyitja
- **THEN** a rendszer nem jelenít meg nyers adóazonosítót egyetlen sorban vagy exportban sem

### Requirement: Teljes audit napló
A rendszer SHALL minden lényegi műveletet naplózzon (ki, mi, mikor), és az audit adatot korlátlanul megőrizze.

#### Scenario: Státuszváltás naplózása
- **WHEN** egy tétel vagy időszak státusza megváltozik
- **THEN** a rendszer naplóbejegyzést ír forrással, időponttal és érintett entitással

#### Scenario: Token-elfogadás jelölése
- **WHEN** egy tétel token-alapú elfogadással kap jóváhagyást
- **THEN** a rendszer a naplóbejegyzést a token-azonosítóhoz köti
- **THEN** a rendszer a bejegyzést gyenge azonosításúként jelöli
```

## Output — Agile Epic file

`openspec/backlog/epics/EPIC-belso-hozzaferes.md`

```markdown
# Belső hozzáférés — M365 auth, táblázat, audit · EPIC-belso-hozzaferes

## Epic
A belső kör (admin és HR/vezetők) M365-tel (Entra ID / OIDC) lép be. Két
szerepkör van: `admin` (szerkeszt: feltöltés, override, lezárás) és `reader`
(csak olvas). A belső felhasználók táblázatként látják az időszak adatait,
szűrhetik és aggregálhatják. Minden lényegi művelet naplózódik (ki, mi, mikor),
és az audit adat korlátlanul megmarad.

## Persona
- Admin (Laci) — `admin` szerepkör, szerkeszt.
- HR / vezetők — `reader` szerepkör, olvas és szűr.
- Adatvédelmi felelős — korlátlan megőrzés jogalapja, adóazonosító-kizárás.

## E2E Scenario
Ha a belső felhasználók M365-tel, szerepkör szerint lépnek be, az adatot
szűrhető/aggregálható táblázatban látják, és minden művelet auditált, akkor
átláthatóvá és visszakövethetővé válik a folyamat az ÁOK vezetése és
adminisztrációja számára, mérve az auditált műveletek lefedettségével és a
szűréssel megválaszolt lekérdezések arányával.

## Problem / Solution
**Probléma:** A mai folyamat nem visszakövethető, és a szűrhető, aggregálható adat helyett
papírlista áll rendelkezésre.

**Megoldás:** Egy M365-alapú, szerepkörös belépés + táblázat-nézet + teljes audit napló
átláthatóvá és elszámoltathatóvá teszi a folyamatot: a vezetők szűrve látják az
adatot, minden státuszváltás pedig visszakereshető — a token-alapú elfogadás
korlátját (gyenge azonosítás) a napló egyértelműen jelöli.

## Cross-cutting Concerns
- Biztonság: szerepkör-szeparáció szerver oldalon kikényszerítve — a `reader`
  nem végezhet szerkesztő műveletet, a megtagadás nem UI-szintű.
- Auditálhatóság: minden lényegi művelet naplózott, az audit adat korlátlanul
  megőrzött.
- Adatkezelés / GDPR: nyers adóazonosító kizárása minden táblázat-nézetből és
  exportból; a korlátlan audit-megőrzés jogalapja tisztázandó.
- Naplózás: minden naplóbejegyzés hordozza a forrást, az időpontot és az
  érintett entitást.
- Identitás: a token-alapú elfogadás gyenge azonosítása a naplóban explicit
  jelölt (token-azonosítóhoz kötve).
- Lokalizáció: a felület és a táblázat-nézet magyar nyelvű.

## MVP and Out of Scope
**MVP:**
- Entra ID (M365 / OIDC) hitelesítés a belső körnek.
- `admin` és `reader` szerepkör; a `reader` nem végezhet szerkesztő műveletet.
- Táblázat-nézet szűréssel és aggregációval, nyers adóazonosító nélkül.
- Teljes audit napló minden lényegi műveletről, korlátlan megőrzéssel.

**Out of Scope:**
- Oktatók / tárgyfelelősök M365-belépése.
- Elosztott szerkesztés.
- Dashboard / statisztikák.
- Automatikus törlő / anonimizáló mechanizmus.

## Success metrics
- Reader-ből indított szerkesztő műveletek 100%-ban megtagadva.
- Lényegi műveletek 100%-a auditált.
- Nyers adóazonosító 0 táblázat-nézetben / exportban.

## Risks and Dependencies
- Függőség: Entra ID tenant + szerepkör-hozzárendelés.
- Függőség: naplózandó események forrásai (`EPIC-excel-ingest`,
  `EPIC-targyfelelos-elfogadas`, `EPIC-idoszak-lezaras`).
- Kockázat (GDPR): korlátlan audit-megőrzés jogalapja.
- Kockázat: token-elfogadás gyenge azonosítása jelölendő.

## High Level Acceptance Criteria
- M365-tel belépő admin eléri a feltöltés / override / lezárás műveleteket.
- Reader szerkesztő művelete megtagadva, adatmódosítás nélkül.
- Táblázat-nézet szűrhető és aggregálható, nyers adóazonosító nélkül.
- Minden státuszváltás naplóbejegyzést kap forrással / időponttal / entitással.
- Token-elfogadás a naplóban token-azonosítóhoz kötött és jelölt.

<!-- pipeline-only:start -->
## Estimation
- **Becsült munkaóra:** 35 ó (≈ 5,8 ideális nap)
- **Story Point:** 18

## Dependency Edges
- depends-on: `EPIC-excel-ingest`
- depends-on: `EPIC-targyfelelos-elfogadas`
- depends-on: `EPIC-idoszak-lezaras`
- external: Entra ID tenant + szerepkör-hozzárendelés

## User Stories
- `STORY-belso-hozzaferes-m365-szerepkorok`
  Mint belső felhasználó, szeretném M365-tel belépni és a szerepkörömnek
  megfelelő jogot kapni, hogy csak az admin szerkeszthessen, a reader pedig
  biztonságosan csak olvasson.
  - Admin M365-belépés után eléri a feltöltés / override / lezárás műveleteket.
  - Reader szerkesztő művelete szerver oldalon megtagadva, adatmódosítás nélkül.
- `STORY-belso-hozzaferes-tablazat-nezet`
  Mint HR/vezető, szeretném az időszak adatait szűrhető, aggregálható
  táblázatban látni nyers adóazonosító nélkül, hogy gyorsan megválaszoljam a
  lekérdezéseket adatvédelmi kockázat nélkül.
  - Átcsúszott tételekre szűrve csak a megfelelő sorok és az aggregált
    összesítés jelenik meg.
  - Nyers adóazonosító sem a nézetben, sem az exportban nem jelenik meg.
- `STORY-belso-hozzaferes-audit-naplo`
  Mint adatvédelmi felelős, szeretném minden lényegi művelet auditált naplóját,
  hogy a folyamat visszakövethető és elszámoltatható legyen.
  - Státuszváltás naplóbejegyzést kap forrással, időponttal és entitással.
  - Token-elfogadás a token-azonosítóhoz kötve, gyenge azonosításúként jelölve.

## Source Spec Reference

| Story ID | Epic ID | Source `### Requirement` | Covered `#### Scenario`s |
|----------|---------|--------------------------|----------------------------|
| STORY-belso-hozzaferes-m365-szerepkorok | EPIC-belso-hozzaferes | M365 alapú belső hozzáférés és szerepkörök | Admin szerkeszthet; Reader csak olvas |
| STORY-belso-hozzaferes-tablazat-nezet | EPIC-belso-hozzaferes | Szűrhető, aggregálható táblázat-nézet | Szűrés átcsúszott tételekre; Nyers adóazonosító elrejtése |
| STORY-belso-hozzaferes-audit-naplo | EPIC-belso-hozzaferes | Teljes audit napló | Státuszváltás naplózása; Token-elfogadás jelölése |
<!-- pipeline-only:end -->
```

## Why these moves

- **English headings, Hungarian body** — every `##` section above the fence uses the new schema's English heading (`Epic`, `Persona`, `E2E Scenario`, `Problem / Solution`, `Cross-cutting Concerns`, `MVP and Out of Scope`, `Success metrics`, `Risks and Dependencies`, `High Level Acceptance Criteria`), while the prose under each stays Hungarian. Even the pipeline-only headings (`User Stories`, `Source Spec Reference`) are English; they never reach Jira, but every heading in the file is English.
- **`## E2E Scenario` absorbs the old Hipotézis** — it carries the measurable hypothesis in the `Ha [X], akkor [Y] a [Z] csoportnak, mérve [W]-vel` shape and keeps the `mérve …` clause, so the epic still ties to a measurable outcome rather than a task list.
- **`## Cross-cutting Concerns` is filled, not rubric text** — the bullets are the real, spec-derived concerns for *this* epic (server-side role separation, unlimited audit retention, adóazonosító exclusion, per-entry provenance, weak-identity token flagging, Hungarian UI). The rubric's menu of possible concern categories is never pasted in; a concern appears only because the spec actually raises it.
- **One lean epic, three Requirements** — all three Requirements serve a single user value (transparent, accountable internal access), so they group into one epic. Three vertical-slice stories, well under the ~10-story ceiling.
- **Requirement-anchored trace IDs** — each `STORY-<epic-slug>-<requirement-slug>` is anchored on its primary Requirement, so the ID stays stable when stories are reordered. No positional numbers.
- **`## Estimation` is a rollup, not an epic-level guess** — the total is the Σ of the child stories' `Eβ` hours and SP (per `ESTIMATION.md`), rendered in the **same two-line shape** as a story card (`Becsült munkaóra:` = `Σ Eβ` ó + ideal days; `Story Point:` = `Σ SP` (bare integer) — no `Σ` prefix, no story-count suffix). It is filled by the estimation-rollup pass *after* `pte-openspec-to-stories` estimates the cards, not at mint. Until every child is estimated it stays a placeholder / `⚠ nem minden story esztimált` — the epic never invents a number its stories don't support.
- **User Stories + Source-Spec map inside the pipeline-only fence** — both sit between `<!-- pipeline-only:start -->` and `<!-- pipeline-only:end -->`, so `pte-openspec-jira-sync` strips the whole region before pushing the Description to Jira, while `pte-openspec-to-stories` and `pte-openspec-bdd-tests` still read the map on disk. The map keeps every `### Requirement` / `#### Scenario` title **verbatim** — that is the cross-skill contract: downstream skills match rows by exact title and source the `WHEN`/`THEN` step text from the spec, never from epic prose.
- **`## Dependency Edges` derived from the narrative** — the three `depends-on` edges are exactly the trace IDs the `## Risks and Dependencies` bullet named as audit-source dependencies (`EPIC-excel-ingest`, `EPIC-targyfelelos-elfogadas`, `EPIC-idoszak-lezaras`), re-expressed as machine-readable edges; the untraced "Entra ID tenant" becomes an `external:` node. It sits inside the same fence (never reaches Jira as text), and it is the single source `pte-openspec-dependency-graph` and `pte-openspec-jira-sync` read — the prose bullet stays human narrative, unparsed.
