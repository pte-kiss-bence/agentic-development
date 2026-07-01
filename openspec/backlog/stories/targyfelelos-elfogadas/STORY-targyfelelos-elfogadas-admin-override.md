# Admin override az elfogadási státuszra · STORY-targyfelelos-elfogadas-admin-override

Parent epic: `EPIC-targyfelelos-elfogadas`

## Description
Mint **admin**, szeretném **bármely elfogadási státuszt kézzel beállítani, ha a tárgyfelelős nem reagál**, hogy **a kifizetés ne akadjon el**.

Az admin bármely `(időszak, tárgy, oktató)` elfogadási státuszt kézzel `elfogadott`-ra állíthat; a rendszer forrásként `admin`-t és az admin kilétét naplózza, és az így elfogadott óra díja a fizetendő összegbe kerül.

## Context
Biztonsági szelep a nem reagáló vagy elérhetetlen tárgyfelelősre — a kifizetési kapu felszabadítása.

## BDD Test
```gherkin
# language: hu
# Spec: targyfelelos-elfogadas › Admin override az elfogadási státuszra (change: aok-osszesito-mvp)
@EPIC-targyfelelos-elfogadas @STORY-targyfelelos-elfogadas-admin-override
Jellemző: targyfelelos-elfogadas
  Az admin kézzel állíthat elfogadási státuszt; ez a kifizetési kaput is felszabadítja.

  Szabály: Admin override az elfogadási státuszra

    Forgatókönyv: Admin beállítja az elfogadást
      Adott egy oktatói sor, amelyet a tárgyfelelős nem fogadott el
      Amikor az admin manuálisan elfogadott-ra állítja
      Akkor a rendszer a státuszt rögzíti forrásként admin-t és az admin kilétét naplózva
      És a sor díja a fizetendő összegbe kerül
```

## Estimation
- **3-Points becslés (ideális óra):** O 3 / M 5 / P 10 | **Eβ 6**, σ 1,2
- **Becsült munkaóra:** 6 ó (≈ 1 ideális nap)
- **Story Point:** 3

## Risks and Dependencies
- Függőség: `STORY-belso-hozzaferes-audit-naplo` (az override naplózása), `STORY-dij-szamitas-fizetendo-elfogadott` (a díj felszabadítása).
- Kockázat: override-visszaélés → a szerepkör- és audit-kontroll kritikus.
