# Historikus migráció — read-only pillanatkép a régi rendszerből · EPIC-historikus-migracio

## Description
Friss start + egyszeri DB→DB migráció: a jelenlegi rendszer adatbázisából a historikus
időszakok adatait read-only pillanatképként töltjük be az új sémába. A régi eredményt
(órák, státuszok, díj, ha van) változatlanul vesszük át — a mai díjmotorral NEM számoljuk
újra. A migráció nem ír a forrásrendszerbe, idempotens, és a személyeket/tárgyakat
ugyanazon kulcsokon (adóazonosító-hash) azonosítja, mint az Excel-ingest. A capability a
mag MVP után fázisolható (a forrás-séma ismerete nyitott).

## Persona
- Admin (Laci) — indítja/felügyeli az egyszeri migrációt, kezeli a migrációs hibalistát.
- HR / vezetők — a historikus időszakokat pillanatképként, olvasva nézik.
- A jelenlegi rendszer gazdája — a forrás-DB sémáját és read-only hozzáférést biztosítja.

## E2E Scenario
Ha a migráció a régi rendszer historikus időszakait read-only pillanatképként betölti az
új sémába, akkor a historikus és az új adat konzisztensen, ugyanazon kulcsokon együtt
nézhető az ÁOK vezetése számára, mérve a sikeresen megfeleltetett forrás-rekordok arányával
(a feloldhatatlanok a migrációs hibalistán), újraszámolt historikus díj nélkül.

## Problem / Solution
P: A régi rendszer szerverét le kell cserélni, de a historikus eredményekhez hozzá kell
férni; ezeket nem szabad a mai díjmotorral újraszámolni (más szabályok szerint készültek),
és a forrás-séma ismerete még nyitott.

S: Egyszeri, idempotens, csak-olvasó DB→DB migráció, amely a régi eredményt változatlan
pillanatképként veszi át, az Excel-ingest kulcsaival (adóazonosító-hash) feleltet meg, a
feloldhatatlanokat hibalistára teszi — és a mag MVP után fázisolható.

## Cross-cutting Concerns
- Adatkezelés: a forrásrendszer csak olvasásra elérhető; a migráció MUST NOT írjon oda.
- Idempotencia: kétszeri futtatás azonos eredményt ad, duplikátum nélkül.
- Adatintegritás: a historikus díj a forrásból változatlan; a mai díjmotor nem fut rá.
- Kulcs-konzisztencia: személy/tárgy azonosítás ugyanazon kulcsokon, mint az Excel-ingest
  (adóazonosító-hash), hogy a historikus és az új adat összeálljon.
- Hibakezelés: a feloldhatatlan forrás-rekord migrációs hibalistára kerül, hibás
  összerendelés nélkül.
- Fázisolhatóság: a forrás-séma nyitott (Open Question B) — a capability a mag MVP után is
  szállítható.

## MVP and Out of Scope
MVP:
- Egyszeri, idempotens, read-only DB→DB migráció a historikus időszakokra.
- A régi eredmény (óra, státusz, díj) változatlan átvétele, újraszámolás nélkül.
- Kulcs-megfeleltetés az Excel-ingest kulcsaihoz (adóazonosító-hash).
- Migrációs hibalista a feloldhatatlan forrás-rekordokra.

Out of Scope:
- Régi Excel-formátumok migrációja (csak a meglévő DB-ből, pillanatképként).
- A historikus díj újraszámítása a mai motorral.
- Írás a forrásrendszerbe / kétirányú szinkron.

## Success metrics
- A feloldható forrás-rekordok 100%-a pillanatképként betöltve, 0 újraszámolt historikus
  díjjal.
- Kétszeri futtatás után 0 duplikált rekord.
- A feloldhatatlan forrás-rekordok 100%-a a migrációs hibalistán (0 hibás összerendelés).

## Risks and Dependencies
- Függőség: a jelenlegi rendszer forrás-DB sémája (Open Question B) — enélkül feltételes.
- Függőség: read-only hozzáférés a forrásrendszerhez.
- Függőség: `EPIC-excel-ingest` kulcs-konvenciója (adóazonosító-hash).
- Kockázat: a forrás-séma ismeretének hiánya miatt a capability fázisolható a mag MVP után.

## High Level Acceptance Criteria
- A régi, lezárt időszak díja változatlan pillanatképként átveendő; a mai díjmotor nem fut rá.
- Ahol a forrás nem tárolt díjat, a historikus nézet a meglévő adatot mutatja, kitalált díj
  nélkül.
- Kétszeri futtatás azonos eredmény, duplikátum nélkül (idempotens).
- Feloldhatatlan kulcsú forrás-rekord a migrációs hibalistára kerül, hibás összerendelés
  nélkül.

## Estimation
- **Becsült munkaóra:** 29 ó (≈ 4,8 ideális nap)
- **Story Point:** 13

<!-- pipeline-only:start -->
## User story-k
- `STORY-historikus-migracio-pillanatkep-import`
  Mint HR/vezető, szeretném a historikus időszakokat read-only pillanatképként az új
  rendszerben látni, hogy a régi eredmények elérhetők maradjanak újraszámolás nélkül.
  - Régi, lezárt időszak díja változatlan pillanatképként átvéve, a mai díjmotor nélkül.
  - Ahol a forrás nem tárolt díjat, a nézet a meglévő adatot mutatja, kitalált díj nélkül.
  - Kétszeri futtatás azonos eredmény, duplikátum nélkül (idempotens).
- `STORY-historikus-migracio-kulcs-megfeleltetes`
  Mint admin, szeretném a migrált személyeket/tárgyakat az Excel-ingest kulcsaival
  azonosítani és a feloldhatatlanokat hibalistán látni, hogy a historikus és az új adat
  konzisztensen összeálljon.
  - Feloldhatatlan kulcsú forrás-rekord a migrációs hibalistára kerül, hibás összerendelés
    nélkül.

## Forrás-spec hivatkozás

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| STORY-historikus-migracio-pillanatkep-import | EPIC-historikus-migracio | Historikus adat importja változtathatatlan pillanatképként | Historikus díj nem számolódik újra; Régi rendszer nem tárolt díjat; Idempotens újrajátszás |
| STORY-historikus-migracio-kulcs-megfeleltetes | EPIC-historikus-migracio | Migrált adat megfeleltetése az új kulcsokhoz | Feloldhatatlan forrás-rekord jelölése |
<!-- pipeline-only:end -->
