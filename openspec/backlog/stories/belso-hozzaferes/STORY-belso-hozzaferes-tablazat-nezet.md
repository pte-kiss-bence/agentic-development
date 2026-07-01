# Díjas táblázat-nézet szűréssel és aggregációval · STORY-belso-hozzaferes-tablazat-nezet

Parent epic: `EPIC-belso-hozzaferes`

## Description
Mint **HR/vezető**, szeretném **az időszak adatait díjjal, szűrhető/aggregálható táblázatban látni nyers adóazonosító nélkül**, hogy **gyorsan megválaszoljam a lekérdezéseket adatvédelmi kockázat nélkül**.

A táblázat oktató × tárgy × óra × díj bontásban jelenik meg, szűréssel (elfogadott/átcsúszott, függő díj, saját-elfogadás, szerződés-státusz, kurzusnyelv, tárgyfelelős) és aggregációval; a függő díj korosítva (aging) látszik; nyers adóazonosító nincs.

## Context
A belső kör fő munkafelülete, amely a papírlistát váltja ki, és a jövőbeli bérszámfejtési exportot is előkészíti.

## BDD Test
```gherkin
# language: hu
# Spec: belso-hozzaferes › Táblázat-nézet díjjal, szűréssel és aggregációval (change: aok-osszesito-mvp)
@EPIC-belso-hozzaferes @STORY-belso-hozzaferes-tablazat-nezet
Jellemző: belso-hozzaferes
  Díjas táblázat-nézet szűréssel, aggregációval, korosított függő díjjal, adóazonosító nélkül.

  Szabály: Táblázat-nézet díjjal, szűréssel és aggregációval

    Forgatókönyv: Szűrés átcsúszottakra
      Adott egy belső felhasználó a táblázat-nézetben
      Amikor az „átcsúszott" szűrőt alkalmazza
      Akkor a táblázat csak a nem_elfogadott / átcsúszott (függő díjú) sorokat mutatja

    Forgatókönyv: Függő díj korosítva
      Adott egy belső felhasználó a táblázat-nézetben
      Amikor a függő díjakat nézi
      Akkor a rendszer korosítva mutatja, mennyi ideje függő az adott tétel

    Forgatókönyv: Saját-elfogadás szűrése
      Adott egy belső felhasználó a táblázat-nézetben
      Amikor a „saját-elfogadás" szűrőt alkalmazza
      Akkor a táblázat csak azokat a sorokat mutatja, ahol a tárgyfelelős a saját óráit fogadta el

    Forgatókönyv: Aggregált díj oktatónként
      Adott egy belső felhasználó a táblázat-nézetben
      Amikor oktatónként aggregál
      Akkor a rendszer oktatónként összesíti a fizetendő és a függő díjat
      És nem jelenít meg nyers adóazonosítót
```

## Estimation
- **3-Points becslés (ideális óra):** O 10 / M 15 / P 30 | **Eβ 17**, σ 3,3
- **Becsült munkaóra:** 17 ó (≈ 2,8 ideális nap)
- **Story Point:** 8

## Risks and Dependencies
- Függőség: `EPIC-dij-szamitas` (a díjadat), `STORY-belso-hozzaferes-m365-szerepkorok` (a hozzáférés), `STORY-targyfelelos-elfogadas-sajat-elfogadas` (a self-approval jelző).
- Kockázat: nyers adóazonosító kiszivárgása a nézetben vagy az exportban.
