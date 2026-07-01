# Lezárt időszak sérthetetlensége · STORY-idoszak-lezaras-immutabilitas

Parent epic: `EPIC-idoszak-lezaras`

## Description
Mint **auditor**, szeretném **a lezárt időszakot változtathatatlannak**, hogy **a késői sor se módosíthassa az aggregátumot vagy a PDF-et**.

A lezárt időszak aggregátumai és a generált PDF nem változhatnak. A lezárt hónapba eső, később érkező sor nem módosítja a lezárt időszakot, hanem admin-döntésre (kiigazítás vagy elvetés) kerül, a lezárt időszak érintetlenül hagyásával, auditáltan.

## Context
Az elszámolás megbízhatóságának alapja: a lezárt hónap befagy (ADR 0002).

## BDD Test
```gherkin
# language: hu
# Spec: idoszak-lezaras › Lezárt időszak sérthetetlensége (change: aok-osszesito-mvp)
@EPIC-idoszak-lezaras @STORY-idoszak-lezaras-immutabilitas
Jellemző: idoszak-lezaras
  A lezárt időszak aggregátuma és PDF-je nem változik; késői sor admin-döntésre.

  Szabály: Lezárt időszak sérthetetlensége

    Forgatókönyv: Késői sor nem módosítja a lezárt időszakot
      Adott egy már lezárt hónaphoz tartozó időszak
      Amikor egy későbbi feltöltésben egy ebbe a hónapba eső sor érkezik
      Akkor a rendszer a lezárt időszakot nem módosítja
      És a sort admin-döntésre jelöli
      És a döntést auditálja
```

## Estimation
- **3-Points becslés (ideális óra):** O 5 / M 8 / P 16 | **Eβ 9**, σ 1,8
- **Becsült munkaóra:** 9 ó (≈ 1,5 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: `STORY-excel-ingest-idoszak-particionalas` (a lezárt hónapba eső sor felismerése), `STORY-idoszak-lezaras-kiigazitas` (a késői sor rendezési útja).
- Kockázat: az immutabilitás résein átszivárgó módosítás → auditálhatatlan elszámolás.
