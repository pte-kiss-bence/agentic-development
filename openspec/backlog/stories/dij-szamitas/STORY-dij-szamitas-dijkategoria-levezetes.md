# Díjkategória és tétel levezetése referenciatáblából · STORY-dij-szamitas-dijkategoria-levezetes

Parent epic: `EPIC-dij-szamitas`

## Description
Mint **admin**, szeretném **a díjkategóriát karbantartott referenciatáblából levezetni a testnevelési/nyelvi kivételekkel**, hogy **a tétel helyes legyen ott is, ahol az oszlopok nem árulják el**.

A tétel a `Kurzustípus`-ból jön (előadás 21 000, szeminárium/gyakorlat 10 500), a testnevelési/nyelvi kategória 7 200 felülírással. A besorolás egy karbantartott, tárgy-szintű referenciatáblából (Szervezeti egység, illetve Nyelvi Intézeten belül Tárgykód-kivétellista) származik, NEM oszlop-heurisztikából. Eldönthetetlen kategória admin-döntésre.

## Context
Kritikus különbségtétel: a Nyelvi Intézeten a 7 200 és a 10 500 minden strukturált mezőben azonos (ADR 0003) — emberi/táblás döntés kell.

## BDD Test
```gherkin
# language: hu
# Spec: dij-szamitas › Díjkategória és tétel levezetése (change: aok-osszesito-mvp)
@EPIC-dij-szamitas @STORY-dij-szamitas-dijkategoria-levezetes
Jellemző: dij-szamitas
  A tétel referenciatáblából, a testnevelési/nyelvi kivételekkel; eldönthetetlen admin-döntésre.

  Szabály: Díjkategória és tétel levezetése

    Forgatókönyv: Testnevelési gyakorlat kedvezményes tétele
      Adott egy Gyakorlat sor, amelynek Szervezeti egysége a besoroló táblában testnevelési egységként szerepel
      Amikor a rendszer levezeti a díjkategóriát
      Akkor a tételt 7 200 Ft/tanórára állítja, nem 10 500-ra

    Forgatókönyv: Nyelvi Intézeten belüli szaktantárgy
      Adott egy Nyelvi Intézethez tartozó sor, amelynek Tárgykódja a kivétellistán szaktantárgyként szerepel
      Amikor a rendszer levezeti a díjkategóriát
      Akkor a normál kurzustípus-tételt alkalmazza (pl. 10 500), nem a 7 200-as nyelvi tételt

    Forgatókönyv: Eldönthetetlen kategória admin-döntésre
      Adott egy Nyelvi Intézeti sor, amelynek Tárgykódja nincs a besoroló táblában
      Amikor a rendszer levezeti a díjkategóriát
      Akkor a sort admin-döntésre jelöli választható kategóriával
      És a díjat addig nem véglegesíti
```

## Estimation
- **3-Points becslés (ideális óra):** O 8 / M 12 / P 22 | **Eβ 13**, σ 2,3
- **Becsült munkaóra:** 13 ó (≈ 2,2 ideális nap)
- **Story Point:** 8

## Risks and Dependencies
- Függőség: a besoroló referenciatábla + adatgazda (Nyelvi Intézet szaktantárgy-listája, Open Question F), `STORY-belso-hozzaferes-admin-feloldas` (az eldönthetetlen feloldása).
- Kockázat: a referenciatábla visszaegyszerűsítése oszlop-levezetéssé — a Nyelvi Intézeten elhasal (ADR 0003).
