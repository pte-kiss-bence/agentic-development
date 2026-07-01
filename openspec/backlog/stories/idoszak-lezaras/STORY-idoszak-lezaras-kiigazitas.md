# Lezárás utáni függő díj rendezése kiigazítással · STORY-idoszak-lezaras-kiigazitas

Parent epic: `EPIC-idoszak-lezaras`

## Description
Mint **oktató**, szeretném **a lezárás után felszabadított függő díjamat a következő időszak kiigazításaként megkapni**, hogy **a lezárt hónap érintetlen maradjon**.

Egy lezárt időszakban átcsúszottként maradt függő díj később rendeződhet anélkül, hogy a lezárt időszakot módosítaná: a felszabadított díj kiigazítás tételként a következő nyitott időszakban jelenik meg, az eredeti `(időszak, tárgy, oktató)` tételre hivatkozva. Mivel lezáráskor minden token érvénytelen, a rendezés kizárólag admin override útján történik (nincs új token).

## Context
A késés utólagos, auditált rendezése a következő hónapban — a lezárt PDF és aggregátum befagyva (ADR 0006).

## BDD Test
```gherkin
# language: hu
# Spec: idoszak-lezaras › Lezárás utáni függő díj rendezése kiigazítással (change: aok-osszesito-mvp)
@EPIC-idoszak-lezaras @STORY-idoszak-lezaras-kiigazitas
Jellemző: idoszak-lezaras
  A felszabadított függő díj kiigazításként a következő nyitott időszakban, admin override-dal.

  Szabály: Lezárás utáni függő díj rendezése kiigazítással

    Forgatókönyv: Utólag felszabadított függő díj kiigazításként
      Adott egy lezárt időszak átcsúszott sora függő díjjal
      Amikor az admin utólag override-dal elfogadja
      Akkor a rendszer a lezárt időszakot érintetlenül hagyja
      És a felszabadított díjat kiigazítás tételként a következő nyitott időszakban rögzíti az eredeti tételre hivatkozva, auditáltan

    Forgatókönyv: Lezárás után nincs token
      Adott egy már lezárt időszak tétele
      Amikor egy tárgyfelelős szeretné elfogadni
      Akkor a rendszer nem ad ki új tokent
      És a rendezés csak admin override-dal lehetséges
```

## Estimation
- **3-Points becslés (ideális óra):** O 6 / M 10 / P 20 | **Eβ 11**, σ 2,3
- **Becsült munkaóra:** 11 ó (≈ 1,8 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: `STORY-idoszak-lezaras-immutabilitas` (a befagyott időszak), `STORY-targyfelelos-elfogadas-admin-override` (a felszabadítás), `STORY-dij-szamitas-fizetendo-elfogadott` (a függő díj).
- Kockázat: a kiigazítás és az eredeti tétel közti hivatkozás elvesztése → követhetetlen utólagos kifizetés.
