# A 2 HR-Excel feltöltése és JOIN-ja · STORY-excel-ingest-feltoltes-join

Parent epic: `EPIC-excel-ingest`

## Description
Mint **admin**, szeretném **a 2 HR-Excelt együtt feltölteni és az adóazonosítón JOIN-oltatni**, hogy **a foglalkozás-sorokból szerződés-státusszal és e-maillel ellátott `assignment` rekordok álljanak elő**.

A rendszer az óranyilvántartást és a szerződés-listát a személy adóazonosító-hash kulcsán kapcsolja össze (óranyilvántartás `Adóazonosító` = szerződés `adószám`), és a feloldható sorokból `assignment` rekordot képez. A tárgy adata teljes egészében az óranyilvántartásból származik.

## Context
Ez a díjszámítás bemenete: a két forrás egyesítése egy feloldott, oktató- és tárgy-adattal teli sorhalmazzá.

## BDD Test
```gherkin
# language: hu
# Spec: excel-ingest › A két HR-Excel feltöltése és JOIN-ja (change: aok-osszesito-mvp)
@EPIC-excel-ingest @STORY-excel-ingest-feltoltes-join
Jellemző: excel-ingest
  A 2 HR-Excel feltöltése és adóazonosító-kulcsú JOIN-ja assignment rekordokká.

  Szabály: A két HR-Excel feltöltése és JOIN-ja

    Forgatókönyv: Sikeres feltöltés és JOIN
      Adott két érvényes HR-Excel: az óranyilvántartás és a szerződés-lista
      Amikor az admin együtt feltölti a két fájlt
      Akkor a rendszer az óranyilvántartás sorait a szerződés-adattal az adóazonosító-hash kulcson JOIN-olja
      És a feloldható sorokat assignment rekordként rögzíti
```

## Estimation
- **3-Points becslés (ideális óra):** O 14 / M 18 / P 36 | **Eβ 20**, σ 3,7
- **Becsült munkaóra:** 20 ó (≈ 3,3 ideális nap)
- **Story Point:** 8

## Risks and Dependencies
- Függőség: az óranyilvántartás és a szerződés-lista oszlop-sémája (JOIN-mezők, kötelező oszlopok).
- Kapcsolódás: `STORY-excel-ingest-adoazonosito-vedelem` (a hash-kulcs képzése), `STORY-excel-ingest-validacio-karanten` (a feloldhatatlan sorok).
- Kockázat: ~65 000 soros Excel szerveroldali feldolgozásának teljesítménye.
