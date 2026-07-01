# Eldönthetetlen sorok admin-feloldása a felületen · STORY-belso-hozzaferes-admin-feloldas

Parent epic: `EPIC-belso-hozzaferes`

## Description
Mint **admin**, szeretném **az admin-döntésre jelölt sorokat a felületen, választható opcióval feloldani**, hogy **a döntés auditáltan eltárolódjon és ne kelljen újra megkérdezni**.

A felület megjeleníti az admin-döntésre jelölt sorokat (díjkategória, státusz-ütközés, párosítatlan oktató, lezárt hónapba eső késői sor), soronként választható opcióval; a döntést auditáltan eltárolja, hogy ismételt feldolgozáskor ne kelljen újra megkérdezni, és nem alapértelmez csendben.

## Context
Az „eldönthetetlent az ember dönti el" elv felülete — a döntés perzisztens és auditált.

## BDD Test
```gherkin
# language: hu
# Spec: belso-hozzaferes › Eldönthetetlen sorok admin-feloldása a felületen (change: aok-osszesito-mvp)
@EPIC-belso-hozzaferes @STORY-belso-hozzaferes-admin-feloldas
Jellemző: belso-hozzaferes
  Az admin a felületen, választható opcióval oldja fel az eldönthetetlen sorokat, auditáltan.

  Szabály: Eldönthetetlen sorok admin-feloldása a felületen

    Forgatókönyv: Admin feloldja a díjkategóriát
      Adott egy sor, amely eldönthetetlen díjkategória miatt admin-döntésre vár
      Amikor az admin a felületen kiválasztja a helyes kategóriát
      Akkor a rendszer a döntést auditáltan eltárolja
      És a sor díja véglegesíthetővé válik
```

## Estimation
- **3-Points becslés (ideális óra):** O 5 / M 8 / P 16 | **Eβ 9**, σ 1,8
- **Becsült munkaóra:** 9 ó (≈ 1,5 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: `STORY-excel-ingest-validacio-karanten` és `STORY-dij-szamitas-dijkategoria-levezetes` (az admin-döntésre jelölés forrásai), `STORY-belso-hozzaferes-audit-naplo`.
- Kockázat: a tárolt döntés helytelen újraalkalmazása egy későbbi feltöltésnél.
