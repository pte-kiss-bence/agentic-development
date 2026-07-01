# Párosítatlan oktató kezelése · STORY-dij-szamitas-parositatlan-oktato

Parent epic: `EPIC-dij-szamitas`

## Description
Mint **HR**, szeretném **a szerződés-párt nem találó oktatókat forrás-hiányként listázva látni**, hogy **pótolhassam a hiányzó státuszt**.

A szerződés-fájlban párt nem találó (státusz nélküli) oktató sorai admin-döntésre kerülnek, a díjuk függőben marad, és a rendszer sem fizetettnek, sem nem-fizetettnek nem sorolja be alapértelmezetten; a hiányt a HR felé listázza.

## Context
Adathiány-kezelés: a hiányzó szerződés-pár nem tippelt jogosultság, hanem HR-feladat.

## BDD Test
```gherkin
# language: hu
# Spec: dij-szamitas › Párosítatlan oktató kezelése (change: aok-osszesito-mvp)
@EPIC-dij-szamitas @STORY-dij-szamitas-parositatlan-oktato
Jellemző: dij-szamitas
  A szerződés-párt nem találó oktató admin-döntésre, díja függőben, HR-hiánylistán.

  Szabály: Párosítatlan oktató kezelése

    Forgatókönyv: Nincs szerződés-pár
      Adott egy tanító oktató, akinek adóazonosítója nem oldható fel a szerződés-fájlból
      Amikor a rendszer feldolgozza a sorát
      Akkor a sort admin-döntésre jelöli
      És a díjat függőben tartja
      És felveszi a HR-nek szánt hiánylistába
```

## Estimation
- **3-Points becslés (ideális óra):** O 3 / M 5 / P 10 | **Eβ 6**, σ 1,2
- **Becsült munkaóra:** 6 ó (≈ 1 ideális nap)
- **Story Point:** 3

## Risks and Dependencies
- Függőség: `STORY-dij-szamitas-statusz-feloldas` (a feloldás kimenete), `STORY-excel-ingest-validacio-karanten` (az admin-döntési jelölés).
- Kockázat: a mintában ~11,8% oktatónak nincs szerződés-párja → nagy hiánylista.
