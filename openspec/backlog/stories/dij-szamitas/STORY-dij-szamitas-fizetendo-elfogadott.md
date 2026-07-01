# Fizetendő díj csak elfogadott órákból · STORY-dij-szamitas-fizetendo-elfogadott

Parent epic: `EPIC-dij-szamitas`

## Description
Mint **admin**, szeretném, **hogy a fizetendő végösszeg csak az elfogadott órákat tartalmazza, a nem elfogadottakat pedig függőként**, hogy **a kifizetési kapu érvényesüljön**.

A fizetendő (végleges) díjba csak az elfogadott (token vagy admin override) órák díja számít; a nem elfogadott — lezáráskor átcsúszott — órák díja függő tételként, a fizetendőtől elkülönítve jelenik meg. A függő díj határidő nélkül függő marad; a díjmodell bővíthetőre tervezett, de MVP-ben csak foglalkozás-alapú.

## Context
A díjszámítás és az elfogadás találkozása: a pénz csak igazolt óra után válik véglegessé (ADR 0004).

## BDD Test
```gherkin
# language: hu
# Spec: dij-szamitas › A fizetendő díj csak elfogadott órákból (change: aok-osszesito-mvp)
@EPIC-dij-szamitas @STORY-dij-szamitas-fizetendo-elfogadott
Jellemző: dij-szamitas
  Fizetendő csak elfogadott órákból; a nem elfogadott díja függő tétel.

  Szabály: A fizetendő díj csak elfogadott órákból

    Forgatókönyv: Átcsúszott óra díja függő
      Adott egy oktató (tárgy × oktató) sora, amely az időszak lezárásakor nem elfogadott
      Amikor a rendszer meghatározza a fizetendő díjat
      Akkor a sor díját függő tételként tartja
      És nem számítja a fizetendő végösszegbe

    Forgatókönyv: Admin override felszabadítja a díjat
      Adott egy korábban nem elfogadott sor
      Amikor az admin override-dal elfogadja
      Akkor a rendszer a sor díját a fizetendő végösszegbe sorolja
```

## Estimation
- **3-Points becslés (ideális óra):** O 5 / M 8 / P 16 | **Eβ 9**, σ 1,8
- **Becsült munkaóra:** 9 ó (≈ 1,5 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: `EPIC-targyfelelos-elfogadas` (az elfogadási státusz + admin override), `EPIC-idoszak-lezaras` (az átcsúszott-jelölés).
- Kockázat: a függő/fizetendő elkülönítés téves aggregálása → hibás kifizetési végösszeg.
