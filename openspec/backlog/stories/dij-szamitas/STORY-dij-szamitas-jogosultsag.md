# Jogosultság státusz és kurzusnyelv szerint · STORY-dij-szamitas-jogosultsag

Parent epic: `EPIC-dij-szamitas`

## Description
Mint **admin**, szeretném **a fizetési jogosultságot státusz × kurzusnyelv szerint eldönteni**, hogy **a munkaköri kötelezettségként tartott magyar órák ne fizetődjenek ki tévesen**.

`EÜ szolgálati jogviszony` vagy `MÁS (nem PTE) alkalmazott` státusznál minden nyelvű óra jogosult. Minden más státusznál csak az idegennyelvű (nem magyar) óra jogosult; a magyar nyelvű óra díja 0 Ft.

## Context
A jogosultság dönti el, hogy egy megtartott óra egyáltalán fizetett-e.

## BDD Test
```gherkin
# language: hu
# Spec: dij-szamitas › Jogosultság szerződéstípus és kurzusnyelv szerint (change: aok-osszesito-mvp)
@EPIC-dij-szamitas @STORY-dij-szamitas-jogosultsag
Jellemző: dij-szamitas
  A fizetési jogosultság a szerződés-státusz × kurzusnyelv szabály szerint.

  Szabály: Jogosultság szerződéstípus és kurzusnyelv szerint

    Forgatókönyv: Közalkalmazott magyar órája nem fizetett
      Adott egy ÁOK közalkalmazott státuszú oktató sora, amelynek Kurzus nyelve magyar
      Amikor a rendszer eldönti a jogosultságot
      Akkor a sor díját 0 Ft-ra állítja

    Forgatókönyv: Közalkalmazott idegennyelvű órája fizetett
      Adott ugyanezen oktató sora, amelynek Kurzus nyelve angol vagy német
      Amikor a rendszer eldönti a jogosultságot
      Akkor a sort jogosultként kezeli
      És kiszámolja a díjat

    Forgatókönyv: EÜ szolgálati jogviszony minden nyelven fizetett
      Adott egy EÜ szolgálati jogviszony státuszú oktató magyar nyelvű órája
      Amikor a rendszer eldönti a jogosultságot
      Akkor a sort jogosultként kezeli
```

## Estimation
- **3-Points becslés (ideális óra):** O 6 / M 9 / P 16 | **Eβ 10**, σ 1,7
- **Becsült munkaóra:** 10 ó (≈ 1,7 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: `STORY-dij-szamitas-statusz-feloldas` (a feloldott státusz), `EPIC-excel-ingest` (`Kurzus nyelve`).
- Kockázat: a státusz-értékek nem szabványos szövegezése a szerződés-fájlban → téves jogosultság.
