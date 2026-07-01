# Foglalkozás-alapú díjszámítás tanórában · STORY-dij-szamitas-foglalkozas-alapu-szamitas

Parent epic: `EPIC-dij-szamitas`

## Description
Mint **oktató**, szeretném, **hogy a díjam soronként `tanóra × tétel` alapon, minden megtartott alkalmamra teljes díjjal számolódjon**, hogy **a társtanított óráim is hiánytalanul elszámoljanak**.

A díj soronként `tanóra × kategória-tétel`, ahol `tanóra = Órahossz_perc / 45`; az oktatóra jutó díj az időszakon belüli sorai összege. Csak a megtartott sorok (`Nemindul` = Hamis) számítanak. Minden sor önálló, díjazott alkalom: társtanításnál minden oktatói sor teljes díjat kap, slot-osztás nélkül.

## Context
A rendszer valódi terméke: a korrekt, soronként visszavezethető oktatói díj.

## BDD Test
```gherkin
# language: hu
# Spec: dij-szamitas › Foglalkozás-alapú díjszámítás tanórában (change: aok-osszesito-mvp)
@EPIC-dij-szamitas @STORY-dij-szamitas-foglalkozas-alapu-szamitas
Jellemző: dij-szamitas
  Soronkénti, tanóra-egységű díjszámítás; társtanításnál minden sor teljes díj.

  Szabály: Foglalkozás-alapú díjszámítás tanórában

    Forgatókönyv: 90 perces előadás díja
      Adott egy megtartott előadás-sor, amelynek Órahossza 90 perc
      Amikor a rendszer kiszámítja a sor díját
      Akkor 2 tanórát számol (90/45)
      És a sor díja 2 × 21 000 = 42 000 Ft

    Forgatókönyv: Meg nem tartott sor kihagyása
      Adott egy sor, amelynek Nemindul értéke nem „Hamis"
      Amikor a rendszer kiszámítja a díjat
      Akkor a sort nem számítja be a díjba

    Forgatókönyv: Társtanított foglalkozás — mindenki teljes díj
      Adott ugyanahhoz a (kurzus, időpont) foglalkozáshoz több oktató külön-külön sorként
      Amikor a rendszer kiszámítja a díjat
      Akkor minden sort önálló oktatási alkalomként, teljes díjjal számol
      És nem oszt slot-szinten
```

## Estimation
- **3-Points becslés (ideális óra):** O 8 / M 13 / P 26 | **Eβ 15**, σ 3
- **Becsült munkaóra:** 15 ó (≈ 2,5 ideális nap)
- **Story Point:** 8

## Risks and Dependencies
- Függőség: `STORY-dij-szamitas-dijkategoria-levezetes` (a tétel), `STORY-dij-szamitas-jogosultsag` (fizetett-e), `EPIC-excel-ingest` (a foglalkozás-sorok).
- Kockázat: a slot-szintű osztás téves bevezetése (ADR 0007) — ez szándékosan tilos, egy jövőbeli fejlesztő „duplikációnak" nézheti.
