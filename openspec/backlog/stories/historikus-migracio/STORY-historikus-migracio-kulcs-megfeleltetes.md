# Migrált adat megfeleltetése az új kulcsokhoz · STORY-historikus-migracio-kulcs-megfeleltetes

Parent epic: `EPIC-historikus-migracio`

## Description
Mint **admin**, szeretném **a migrált személyeket/tárgyakat az Excel-ingest kulcsaival azonosítani és a feloldhatatlanokat hibalistán látni**, hogy **a historikus és az új adat konzisztensen összeálljon**.

A migráció a személyeket adóazonosító-hash kulcson, a tárgyakat ugyanazon kulcsokon azonosítja, mint az Excel-ingest, hogy a historikus és az új adat együtt álljon össze. A nem feloldható forrás-rekordokat migrációs hibalistába teszi, hibás összerendelés létrehozása nélkül.

## Context
Kulcs-konzisztencia: a historikus és az új rekordok ugyanazon a személy/tárgy azonosságon nyugszanak.

## BDD Test
```gherkin
# language: hu
# Spec: historikus-migracio › Migrált adat megfeleltetése az új kulcsokhoz (change: aok-osszesito-mvp)
@EPIC-historikus-migracio @STORY-historikus-migracio-kulcs-megfeleltetes
Jellemző: historikus-migracio
  A migrált személyek/tárgyak az Excel-ingest kulcsaival; a feloldhatatlan a hibalistára.

  Szabály: Migrált adat megfeleltetése az új kulcsokhoz

    Forgatókönyv: Feloldhatatlan forrás-rekord jelölése
      Adott egy forrás-rekord, amelynek kulcsa nem feleltethető meg az új séma kulcsainak
      Amikor a migráció feldolgozza
      Akkor a rendszer a rekordot migrációs hibalistába teszi
      És nem hoz létre belőle hibás összerendelést
```

## Estimation
- **3-Points becslés (ideális óra):** O 5 / M 9 / P 24 | **Eβ 11**, σ 3,2
- **Becsült munkaóra:** 11 ó (≈ 1,8 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: `STORY-excel-ingest-adoazonosito-vedelem` (a kulcs-konvenció), `STORY-historikus-migracio-pillanatkep-import` (a migrációs futás).
- Kockázat: a forrás- és az új séma kulcsainak eltérése → nagy migrációs hibalista.
