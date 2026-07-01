# Elfogadás mint kifizetési kapu és audit · STORY-targyfelelos-elfogadas-kifizetesi-kapu

Parent epic: `EPIC-targyfelelos-elfogadas`

## Description
Mint **auditor**, szeretném **az elfogadást kifizetési kapuként és auditáltan kezelni**, hogy **csak az elfogadott órák díja legyen végleges és a token gyenge azonosítása látszódjon**.

A rendszer az elfogadást (token vagy admin) a kifizetés kapujaként kezeli, és az elfogadás metaadatát (időpont, forrás, token esetén technikai jelzők, pl. IP/user-agent) naplózza; az auditban egyértelműen jelzi, hogy a token-elfogadás nem erős személyazonosítás.

## Context
Az elfogadás nemcsak checkpoint, hanem pénzt kapuz — az audit rögzíti a bizonyíték és a kilét korlátját.

## BDD Test
```gherkin
# language: hu
# Spec: targyfelelos-elfogadas › Elfogadás mint kifizetési kapu és audit (change: aok-osszesito-mvp)
@EPIC-targyfelelos-elfogadas @STORY-targyfelelos-elfogadas-kifizetesi-kapu
Jellemző: targyfelelos-elfogadas
  Az elfogadás a kifizetés kapuja; a token-elfogadás gyenge azonosítása auditban jelölt.

  Szabály: Elfogadás mint kifizetési kapu és audit

    Forgatókönyv: Token-elfogadás auditja a kilét korlátjával
      Adott egy token útján elfogadott óra
      Amikor a rendszer naplózza az elfogadást
      Akkor a napló a művelet forrását token-ként, az időponttal és a technikai metaadattal rögzíti
      És jelzi, hogy ez nem erős személyazonosítás
```

## Estimation
- **3-Points becslés (ideális óra):** O 4 / M 6 / P 12 | **Eβ 7**, σ 1,3
- **Becsült munkaóra:** 7 ó (≈ 1,2 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: `STORY-belso-hozzaferes-audit-naplo` (a napló-infrastruktúra), `STORY-targyfelelos-elfogadas-token-nezet` (az elfogadási esemény).
- Kockázat: a gyenge azonosítás jelöletlenül maradása → auditálhatatlan kifizetés-alap.
