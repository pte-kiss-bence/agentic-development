# Áttekintő összesítő PDF visszakereső azonosítóval · STORY-idoszak-lezaras-osszesito-pdf

Parent epic: `EPIC-idoszak-lezaras`

## Description
Mint **HR**, szeretném **a lezáráskori áttekintő PDF-et visszakereső azonosítóval**, hogy **a papírpéldányból a webes részletes táblázat visszakereshető legyen**.

Lezáráskor a rendszer egyetlen áttekintő PDF-et állít elő: aggregált óra- és fizetendő díjösszegek, az elfogadott vs. átcsúszott/függő tételek kiemelése, és egy visszakereső azonosító (időszak-referencia/GUID). A PDF nem tartalmaz nyers adóazonosítót, forintot csak aggregált szinten.

## Context
A papírfolyamat kiváltásának végterméke: egy áttekintő oldal, visszakereshetően.

## BDD Test
```gherkin
# language: hu
# Spec: idoszak-lezaras › Áttekintő összesítő PDF visszakereső azonosítóval (change: aok-osszesito-mvp)
@EPIC-idoszak-lezaras @STORY-idoszak-lezaras-osszesito-pdf
Jellemző: idoszak-lezaras
  Lezáráskor egy áttekintő PDF aggregált óra/díj adatokkal és visszakereső azonosítóval.

  Szabály: Áttekintő összesítő PDF visszakereső azonosítóval

    Forgatókönyv: PDF generálása lezáráskor
      Adott egy lezárás előtt álló időszak aggregált óra- és díjadatokkal
      Amikor az időszak lezárul
      Akkor a rendszer egyetlen áttekintő PDF-et állít elő az aggregált óra- és díjösszegekkel
      És kiemeli az átcsúszott/függő tételeket
      És visszakereső azonosítót tartalmaz
      És nem tartalmaz nyers adóazonosítót
```

## Estimation
- **3-Points becslés (ideális óra):** O 6 / M 10 / P 20 | **Eβ 11**, σ 2,3
- **Becsült munkaóra:** 11 ó (≈ 1,8 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: `STORY-idoszak-lezaras-manualis-lezaras` (a lezárás eseménye), PDF-generáló komponens, `STORY-dij-szamitas-fizetendo-elfogadott` (az aggregált összegek).
- Kockázat: nyers adóazonosító vagy soronkénti forint véletlen kiszivárgása a PDF-ben.
