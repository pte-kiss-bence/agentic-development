# Token-link kiküldése a tárgyfelelősnek · STORY-targyfelelos-elfogadas-token-kikuldes

Parent epic: `EPIC-targyfelelos-elfogadas`

## Description
Mint **admin**, szeretném **az érintett tárgyfelelősöknek egyedi token-linket kiküldeni**, hogy **login nélkül elfogadhassák az órákat, biztonságosan tárolt tokennel**.

Az „Értesítések kiküldése" minden érintett tárgyfelelősnek egyedi, kriptográfiailag erős (≥128 bit) token-linket küld egy `(időszak, tárgyfelelős)` párra; a token csak hash-elve tárolódik, a nyers token csak a linkben szerepel, és az érvényesség a lezárásig tart. A tárgyfelelős e-mail nélkül a „kézbesíthetetlen" listára kerül.

## Context
A papíraláírásos igazolás kiváltásának belépőpontja: a token-link a tárgyfelelőshöz.

## BDD Test
```gherkin
# language: hu
# Spec: targyfelelos-elfogadas › Token-link kiküldése a tárgyfelelősnek (change: aok-osszesito-mvp)
@EPIC-targyfelelos-elfogadas @STORY-targyfelelos-elfogadas-token-kikuldes
Jellemző: targyfelelos-elfogadas
  Egyedi, erős token-link (időszak, tárgyfelelős) páronként; a token csak hash-elve tárolt.

  Szabály: Token-link kiküldése a tárgyfelelősnek

    Forgatókönyv: Értesítés kiküldése
      Adott egy időszak rendezett karantén- és admin-döntéseivel
      Amikor az admin az „Értesítések kiküldése" műveletet indítja
      Akkor a rendszer minden tárgyfelelősnek személyre szabott e-mailt küld egyedi token-linkkel
      És a kiküldést naplózza

    Forgatókönyv: Token csak hash-elve tárolódik
      Adott egy tárgyfelelős, akinek token-rekordot kell létrehozni
      Amikor a rendszer létrehozza a token-rekordot
      Akkor csak a token hash-e kerül tárolásra
      És a nyers token nem visszafejthető az adatbázisból

    Forgatókönyv: Hiányzó tárgyfelelős e-mail
      Adott egy tárgyfelelős, akihez nincs e-mail cím
      Amikor a rendszer kiküldené az értesítést
      Akkor a tételt „kézbesíthetetlen" listára teszi az admin számára
      És az elfogadás admin override útján oldható meg
```

## Estimation
- **3-Points becslés (ideális óra):** O 8 / M 12 / P 30 | **Eβ 15**, σ 3,7
- **Becsült munkaóra:** 15 ó (≈ 2,5 ideális nap)
- **Story Point:** 8

## Risks and Dependencies
- Függőség: az e-mail küldés mechanizmusa (Open Question A) — interfész mögé tervezve; a tárgyfelelős e-mailje az Excelből (Open Question E).
- Kockázat: továbbküldött token-link → jogosulatlan elfogadás (tudatosan vállalva); kompenzáció: ≥128 bit, csak hash tárolva, lezáráskor érvénytelenítve.
