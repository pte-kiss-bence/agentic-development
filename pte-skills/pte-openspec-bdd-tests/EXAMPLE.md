# Worked example: story card → embedded Gherkin

One end-to-end transformation showing the non-obvious moves under the **new lean card schema**: the story card carries **no trace row of its own**, so this skill learns which `#### Scenario`s the card covers — and which tags to apply — by reading the **parent epic's** pipeline-only *Source Spec Reference* map, matched to the card by its `STORY-…`. It then traces those Scenarios back to the spec for the `WHEN`/`THEN` behaviour, supplies the missing `Adott` (Given), writes declarative Gherkin into the card's `## BDD Test` section — in place, touching nothing else.

The pedagogical shape is unchanged: a short `## Input` (the story card with an empty `## BDD Test` placeholder, plus the epic map row it matches), the `## Output` (the same card with `## BDD Test` filled), then a `## Why these moves` notes section. Prose is English; the card's content and the Gherkin are Hungarian (`# language: hu`, Hungarian keywords) per `CONVENTIONS.md`; every trace ID and `### Requirement` / `#### Scenario` title stays **verbatim**.

## Input — the story card (from `pte-openspec-to-stories`)

`openspec/backlog/stories/belso-hozzaferes/STORY-belso-hozzaferes-m365-szerepkorok.md`, with its `## BDD Test` section still a placeholder. Note the lean schema: the card has **no** `## Source Reference` row of its own.

```markdown
# M365 belépés és szerepkörök · STORY-belso-hozzaferes-m365-szerepkorok

## User Story
Mint belső felhasználó, szeretném M365-tel belépni és a szerepkörömnek megfelelő
jogot kapni, hogy csak az admin szerkeszthessen, a reader pedig biztonságosan
csak olvasson.

## Context
A belső kör M365-tel (Entra ID / OIDC) hitelesít. Két szerepkör van: `admin`
(szerkeszt: feltöltés, override, lezárás) és `reader` (csak olvas). Ez a történet
a hitelesítést és a szerepkör-szeparációt szeleteli ki; a táblázat-nézet és az
audit napló külön történet.

## BDD Test
_(kitölti a `pte-openspec-bdd-tests`)_

## Risks and Dependencies
- Függ az Entra ID tenanttól és a szerepkör-hozzárendeléstől.
- A `reader` megtagadása szerver oldalon kényszerítendő ki, nem UI-szinten.

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

The card names no Scenarios itself. To learn its scope this skill reads the **parent epic's** pipeline-only *Source Spec Reference* map in `openspec/backlog/epics/EPIC-belso-hozzaferes.md`, and matches the row whose `Story ID` equals this card's `STORY-…`:

| Story ID | Epic ID | Source `### Requirement` | Covered `#### Scenario`s |
|----------|---------|--------------------------|----------------------------|
| STORY-belso-hozzaferes-m365-szerepkorok | EPIC-belso-hozzaferes | M365 alapú belső hozzáférés és szerepkörök | Admin szerkeszthet; Reader csak olvas |

That row names the two `#### Scenario`s to cover and the trace IDs to tag with. Traced to their `### Requirement` in `openspec/changes/aok-osszesito-mvp/specs/belso-hozzaferes/spec.md`:

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
```

## Output — the same card, `## BDD Test` section filled in place

Only the `## BDD Test` section changes; every other section of the card is left byte-for-byte untouched.

````markdown
## BDD Test
```gherkin
# language: hu
# Spec: belso-hozzaferes › M365 alapú belső hozzáférés és szerepkörök (change: aok-osszesito-mvp)
@EPIC-belso-hozzaferes @STORY-belso-hozzaferes-m365-szerepkorok
Jellemző: belso-hozzaferes
  M365 hitelesítés és szerepkör-szeparáció (admin szerkeszt, reader olvas).

  Szabály: M365 alapú belső hozzáférés és szerepkörök

    Forgatókönyv: Admin szerkeszthet
      Adott egy admin szerepkörű, M365-tel hitelesített felhasználó
      Amikor belép a rendszerbe
      Akkor elérhetők számára a feltöltés, az override és a lezárás műveletek

    Forgatókönyv: Reader csak olvas
      Adott egy reader szerepkörű, M365-tel hitelesített felhasználó
      Amikor egy szerkesztő műveletet próbál indítani
      Akkor a rendszer megtagadja a műveletet
      És nem módosít adatot
```
````

## Why these moves

- **The map is read from the EPIC, not the card** — under the new lean schema the story card carries no trace row. This skill matches the card's `STORY-belso-hozzaferes-m365-szerepkorok` (from its Cím H1 / filename) against the parent `EPIC-belso-hozzaferes`'s pipeline-only *Source Spec Reference* map, and the matched row gives **which** `#### Scenario`s to cover and **which** trace IDs to tag with. The card itself never names them.
- **Spec is the behavioural source** — the map supplies only the Scenario split and the IDs; the `Amikor`/`Akkor` step text is traced back to the `### Requirement`'s `WHEN`/`THEN` bullets in the spec, never lifted from card prose (see `CONVENTIONS.md`).
- **`# language: hu` + Hungarian keywords** — the spec is Hungarian, so the Gherkin keywords (`Jellemző`, `Szabály`, `Forgatókönyv`, `Adott`/`Amikor`/`Akkor`/`És`) come from the gherkin i18n set for that language.
- **Added `Adott` (Given)** — OpenSpec states only the `WHEN` trigger and the `THEN` outcome, never the precondition. Each Scenario's `Adott` is **supplied** here (the authenticated user with a given role) — the one gap between OpenSpec and Gherkin.
- **`Szabály` carries the Requirement, `Jellemző` the capability** — the `Szabály` (Rule) is the source `### Requirement` verbatim; the `Jellemző` (Feature) names the `belso-hozzaferes` capability with a one-line intent.
- **Declarative, no UI detail** — "belép a rendszerbe" / "egy szerkesztő műveletet próbál indítani", not field-level clicks or routes. The behaviour survives a UI redesign; the Playwright/UI detail lives in the later E2E layer, never in the Gherkin.
- **Two Scenarios, not an outline** — "Admin szerkeszthet" and "Reader csak olvas" are two genuinely different behaviours (grant vs. deny), so they stay two `Forgatókönyv`s; only data-only variants collapse into a `Forgatókönyv vázlat`.
- **Tags taken verbatim from the map row** — `@EPIC-belso-hozzaferes` and `@STORY-belso-hozzaferes-m365-szerepkorok` are copied straight from the map, never re-minted (this skill CONSUMES IDs). The Gherkin tags are the card's only in-body trace, so they must match the map exactly.
- **In place, nothing else touched** — only the `## BDD Test` placeholder is replaced; `## User Story`, `## Context`, and `## Risks and Dependencies` stay byte-for-byte.
