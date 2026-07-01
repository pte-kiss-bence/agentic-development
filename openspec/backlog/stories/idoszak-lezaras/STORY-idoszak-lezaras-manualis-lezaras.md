# Manuális lezárás átcsúszott-jelöléssel és függő díjjal · STORY-idoszak-lezaras-manualis-lezaras

Parent epic: `EPIC-idoszak-lezaras`

## Description
Mint **admin**, szeretném **manuálisan lezárni az időszakot akkor is, ha vannak nem elfogadott sorok**, hogy **a HR-késés ne blokkolja a havi zárást**.

Lezáráskor a `nem_elfogadott` (tárgy × oktató) sorok `átcsúszott`-tá válnak, a díjuk függő tételként, a fizetendőtől elkülönítve kezelődik, és minden token érvénytelenné válik. Ha van rendezetlen karantén- vagy admin-döntésre váró sor, a rendszer blokkolóan figyelmeztet, és külön megerősítést kér.

## Context
A zárás nem függ minden tárgyfelelős reakciójától — az átcsúszott/függő út feloldja a késést.

## BDD Test
```gherkin
# language: hu
# Spec: idoszak-lezaras › Manuális lezárás átcsúszott-jelöléssel és függő díjjal (change: aok-osszesito-mvp)
@EPIC-idoszak-lezaras @STORY-idoszak-lezaras-manualis-lezaras
Jellemző: idoszak-lezaras
  Manuális lezárás nem elfogadott sorokkal is; átcsúszott-jelölés, függő díj, token-érvénytelenítés.

  Szabály: Manuális lezárás átcsúszott-jelöléssel és függő díjjal

    Forgatókönyv: Lezárás nem-elfogadottal
      Adott egy időszak, amelyben vannak nem_elfogadott sorok, de nincs rendezetlen karantén
      Amikor az admin lezárja az időszakot
      Akkor a rendszer lezárja az időszakot
      És a nem_elfogadott sorokat átcsúszott-ként jelöli
      És a díjukat függő tételként kezeli
      És minden tokent érvénytelenít

    Forgatókönyv: Figyelmeztetés rendezetlen sorokra
      Adott egy időszak, amelyben van karanténos vagy admin-döntésre váró sor
      Amikor az admin lezárást indít
      Akkor a rendszer blokkoló figyelmeztetést ad
      És a lezáráshoz külön megerősítést kér
```

## Estimation
- **3-Points becslés (ideális óra):** O 8 / M 12 / P 22 | **Eβ 13**, σ 2,3
- **Becsült munkaóra:** 13 ó (≈ 2,2 ideális nap)
- **Story Point:** 8

## Risks and Dependencies
- Függőség: `EPIC-targyfelelos-elfogadas` (elfogadási státusz + tokenek), `STORY-dij-szamitas-fizetendo-elfogadott` (a függő díj), `STORY-excel-ingest-validacio-karanten` (a rendezetlen sorok).
- Kockázat: idő előtti lezárás rendezetlen sorokkal → a blokkoló figyelmeztetés megkerülhetetlensége kritikus.
