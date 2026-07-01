# Több nyitott havi időszak, egyenkénti zárás · STORY-idoszak-lezaras-tobb-nyitott

Parent epic: `EPIC-idoszak-lezaras`

## Description
Mint **admin**, szeretném **egyszerre több nyitott havi időszakot kezelni és egyenként zárni**, hogy **egy több-hónapos feltöltés minden hónapja külön rendezhető legyen**.

Az időszak naptári hónap az `Óra kezdete` szerint; mivel egy feltöltés több hónapot is érinthet, egyszerre több nyitott/vázlat időszak létezhet, és az admin ezeket egyenként, manuálisan zárja.

## Context
A havi elszámolás ritmusa: párhuzamos nyitott hónapok, külön-külön lezárhatóan (ADR 0002).

## BDD Test
```gherkin
# language: hu
# Spec: idoszak-lezaras › Havi időszakok, több nyitott lehet, admin zár egyenként (change: aok-osszesito-mvp)
@EPIC-idoszak-lezaras @STORY-idoszak-lezaras-tobb-nyitott
Jellemző: idoszak-lezaras
  Egyszerre több nyitott havi időszak; az admin egyenként, manuálisan zár.

  Szabály: Havi időszakok, több nyitott lehet, admin zár egyenként

    Forgatókönyv: Több nyitott időszak egy feltöltésből
      Adott egy feltöltés, amely két hónap adatát tölti be
      Amikor a rendszer particionálja az adatot időszakokra
      Akkor mindkét hónaphoz létrejön vagy feltöltődik a nyitott időszak
      És az admin külön-külön zárhatja őket
```

## Estimation
- **3-Points becslés (ideális óra):** O 3 / M 6 / P 12 | **Eβ 7**, σ 1,5
- **Becsült munkaóra:** 7 ó (≈ 1,2 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: `STORY-excel-ingest-idoszak-particionalas` (a hónap-időszakok létrejötte).
- Kockázat: az időszak-életciklus (vázlat/nyitott/lezárt) állapotmodelljének következetessége.
