# Szerződés-státusz feloldása és ütközés kezelése · STORY-dij-szamitas-statusz-feloldas

Parent epic: `EPIC-dij-szamitas`

## Description
Mint **admin**, szeretném **a szerződés-státuszt a hash-kulcson feloldani és az ütközéseket dátum, majd jogosult-preferencia + jelölés szerint rendezni**, hogy **a többes szerződések ne torzítsák a jogosultságot**.

Több, eltérő státuszú szerződésnél az időszakra érvényes dátumú szerződés státusza dönt; ha így is holtverseny marad egy jogosult és egy nem-jogosult státusz között, a rendszer a jogosultat alkalmazza, de a sort admin-jelöléssel felülvizsgálatra jelöli.

## Context
A jogosultság bemenete a helyes státusz; a többes szerződések és ütközések feloldása előfeltétel.

## BDD Test
```gherkin
# language: hu
# Spec: dij-szamitas › Szerződés-státusz feloldása és ütközés kezelése (change: aok-osszesito-mvp)
@EPIC-dij-szamitas @STORY-dij-szamitas-statusz-feloldas
Jellemző: dij-szamitas
  A státusz feloldása a hash-kulcson; ütközésnél dátum, majd jogosult-preferencia + jelölés.

  Szabály: Szerződés-státusz feloldása és ütközés kezelése

    Forgatókönyv: Több szerződés, dátum dönt
      Adott egy személy két, eltérő státuszú szerződéssel, amelyből csak az egyik érvényes az időszak dátumára
      Amikor a rendszer feloldja a szerződés-státuszt
      Akkor az időszakra érvényes szerződés státuszát alkalmazza

    Forgatókönyv: Jogosult és nem-jogosult státusz ütközése
      Adott egy személy, akinél dátum alapján sem oldható fel egy jogosult és egy nem-jogosult státusz ütközése
      Amikor a rendszer feloldja a szerződés-státuszt
      Akkor a jogosult státuszt alkalmazza
      És a sort admin-jelöléssel jelöli felülvizsgálatra
```

## Estimation
- **3-Points becslés (ideális óra):** O 5 / M 8 / P 16 | **Eβ 9**, σ 1,8
- **Becsült munkaóra:** 9 ó (≈ 1,5 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: `STORY-excel-ingest-adoazonosito-vedelem` (a hash-kulcs), a szerződés érvényességi dátum-mezője.
- Kockázat: a mintában 64 személynél jogosult+nem-jogosult státusz keveredik → sok admin-jelölés.
