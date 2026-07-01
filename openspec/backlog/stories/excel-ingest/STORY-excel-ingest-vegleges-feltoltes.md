# Egyszeri, végleges feltöltés időszakonként · STORY-excel-ingest-vegleges-feltoltes

Parent epic: `EPIC-excel-ingest`

## Description
Mint **admin**, szeretném, **hogy egy időszak feltöltése végleges legyen**, hogy **egy felülíró újrafeltöltés ne törölhesse a már rögzített adatot**.

Ugyanazon nyitott hónap-időszakhoz teljes felülíró újrafeltöltés nem lehetséges; a hiányzó/késői sorok az admin-döntési, illetve — lezárt időszaknál — a kiigazítási úton kezelendők.

## Context
Adatvédő korlát: a feltöltött időszak-adat rögzített, nem írható felül véletlenül.

## BDD Test
```gherkin
# language: hu
# Spec: excel-ingest › Egyszeri, végleges feltöltés időszakonként (change: aok-osszesito-mvp)
@EPIC-excel-ingest @STORY-excel-ingest-vegleges-feltoltes
Jellemző: excel-ingest
  Egy nyitott időszak adata a feltöltésből végleges; felülíró újrafeltöltés tiltva.

  Szabály: Egyszeri, végleges feltöltés időszakonként

    Forgatókönyv: Ismételt felülíró feltöltés tiltása
      Adott egy már feltöltött nyitott hónap-időszak
      Amikor az admin ugyanazt az időszakot teljes egészében újra feltöltené
      Akkor a rendszer elutasítja a felülíró feltöltést
      És jelzi, hogy az időszak adata már rögzített
```

## Estimation
- **3-Points becslés (ideális óra):** O 3 / M 5 / P 9 | **Eβ 5**, σ 1
- **Becsült munkaóra:** 5 ó (≈ 0,8 ideális nap)
- **Story Point:** 3

## Risks and Dependencies
- Függőség: `STORY-excel-ingest-idoszak-particionalas` (az időszak-állapot: van-e már feltöltés).
- Kockázat: a „végleges" és a legitim javítási igény közti feszültség → az admin-döntési/kiigazítási út egyértelmű kommunikációja szükséges.
