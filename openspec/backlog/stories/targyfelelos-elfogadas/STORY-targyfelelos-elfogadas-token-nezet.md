# Token-nézet és soronkénti óra-elfogadás · STORY-targyfelelos-elfogadas-token-nezet

Parent epic: `EPIC-targyfelelos-elfogadas`

## Description
Mint **tárgyfelelős**, szeretném **login nélkül, oktatónként elfogadni a tárgyaimon tartott órákat, díj nélkül**, hogy **a papíraláírást kiváltsa egy egyszerű token-nézet**.

A token-linkre érkező tárgyfelelős login nélkül látja a hozzá tartozó tárgyakat, tárgyanként csoportosítva, oktatónként az összesített órákat, soronkénti „Elfogadom"-mal. A nézet forintot és nyers adóazonosítót nem tartalmaz. Az elfogadás végleges (egyirányú); a részleges elfogadás nem blokkolja a többi oktatót; lezárt időszak tokenje érvénytelen.

## Context
A tárgyfelelős egyetlen munkafelülete: csak óra, csak elfogadás, visszavonás nélkül.

## BDD Test
```gherkin
# language: hu
# Spec: targyfelelos-elfogadas › Token-nézet és (tárgy × oktató) soronkénti elfogadás (change: aok-osszesito-mvp)
@EPIC-targyfelelos-elfogadas @STORY-targyfelelos-elfogadas-token-nezet
Jellemző: targyfelelos-elfogadas
  Login nélküli token-nézet, oktatónkénti óra-elfogadás, díj és adóazonosító nélkül.

  Szabály: Token-nézet és (tárgy × oktató) soronkénti elfogadás

    Forgatókönyv: Oktatónkénti elfogadás egy tárgyon
      Adott egy token-nézetet megnyitó tárgyfelelős egy tárgy oktatói órasoraival
      Amikor elfogadja egy oktató óráit az adott tárgyon
      Akkor a rendszer az adott (időszak, tárgy, oktató) státuszát elfogadott-ra állítja
      És forrásként token-t rögzít
      És a sort lezárja további tárgyfelelősi módosítás elől

    Forgatókönyv: Egy vitás oktató nem blokkolja a többit
      Adott egy tárgy több oktatóval a token-nézetben
      Amikor a tárgyfelelős csak néhány oktatót fogad el
      Akkor a rendszer az elfogadott oktatói sorokat elfogadottként rögzíti
      És a többit érintetlenül hagyja

    Forgatókönyv: Díj nem jelenik meg a tárgyfelelősnek
      Adott egy token-nézetet megnyitó tárgyfelelős
      Amikor megtekinti a hozzá tartozó órákat
      Akkor a nézet csak órákat mutat
      És forintösszeget nem

    Forgatókönyv: Lejárt/lezárt időszak tokenje
      Adott egy token, amely egy már lezárt időszakhoz tartozik
      Amikor a tárgyfelelős megnyitja a linket
      Akkor a rendszer érvénytelennek jelzi a linket
      És nem enged elfogadást
```

## Estimation
- **3-Points becslés (ideális óra):** O 10 / M 16 / P 32 | **Eβ 18**, σ 3,7
- **Becsült munkaóra:** 18 ó (≈ 3 ideális nap)
- **Story Point:** 8

## Risks and Dependencies
- Függőség: `STORY-targyfelelos-elfogadas-token-kikuldes` (a token), `EPIC-idoszak-lezaras` (a token-érvényesség határa).
- Kockázat: a díj/adóazonosító véletlen kiszivárgása a token-nézetben vagy az URL-ben.
