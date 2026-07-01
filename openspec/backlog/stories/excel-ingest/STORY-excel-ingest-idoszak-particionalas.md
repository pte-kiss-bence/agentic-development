# Időszak-particionálás az óra dátumából · STORY-excel-ingest-idoszak-particionalas

Parent epic: `EPIC-excel-ingest`

## Description
Mint **admin**, szeretném, **hogy a feltöltött sorok az `Óra kezdete` hónapja szerint a megfelelő havi időszakhoz kerüljenek**, hogy **egy több-hónapos feltöltés is helyesen szétváljon**.

Egy feltöltés több hónapot is tartalmazhat; a rendszer hónap-időszakonként particionál, és minden érintett hónaphoz a nyitott időszakot feltölti vagy létrehozza. A lezárt hónapba eső sort nem a lezárt időszakba írja, hanem admin-döntésre jelöli.

## Context
A havi elszámolás alapegysége az időszak; a particionálás dönti el, melyik sor melyik hónaphoz tartozik.

## BDD Test
```gherkin
# language: hu
# Spec: excel-ingest › Időszak levezetése az óra dátumából, több-hónapos feltöltés (change: aok-osszesito-mvp)
@EPIC-excel-ingest @STORY-excel-ingest-idoszak-particionalas
Jellemző: excel-ingest
  A sorok havi időszakhoz rendelése az Óra kezdete alapján, több-hónapos feltöltésnél.

  Szabály: Időszak levezetése az óra dátumából, több-hónapos feltöltés

    Forgatókönyv: Egy feltöltés több hónapra
      Adott egy feltöltött óranyilvántartás, amely két különböző hónapba eső sorokat tartalmaz
      Amikor a rendszer particionálja a sorokat
      Akkor a sorokat az Óra kezdete hónapja szerint a két külön havi időszakhoz rendeli

    Forgatókönyv: Lezárt hónapba eső sor
      Adott egy már lezárt (immutable) hónaphoz tartozó időszak
      Amikor egy feltöltött sor Óra kezdete-je ebbe a lezárt hónapba esik
      Akkor a rendszer a lezárt időszakot nem módosítja
      És a sort admin-döntésre jelöli
      És a döntést auditálja
```

## Estimation
- **3-Points becslés (ideális óra):** O 12 / M 14 / P 23 | **Eβ 15**, σ 1,8
- **Becsült munkaóra:** 15 ó (≈ 2,5 ideális nap)
- **Story Point:** 8

## Risks and Dependencies
- Függőség: `STORY-excel-ingest-feltoltes-join` (a particionálandó sorok), `EPIC-idoszak-lezaras` (a lezárt időszak állapota).
- Kockázat: időzóna/hónaphatár körüli dátum-élek (`Óra kezdete`) téves besorolása.
