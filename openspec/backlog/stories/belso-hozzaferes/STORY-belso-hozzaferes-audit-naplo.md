# Teljes audit napló · STORY-belso-hozzaferes-audit-naplo

Parent epic: `EPIC-belso-hozzaferes`

## Description
Mint **adatvédelmi felelős**, szeretném **minden lényegi művelet auditált naplóját**, hogy **a folyamat visszakövethető és elszámoltatható legyen**.

A rendszer minden lényegi műveletet naplóz (ingest, admin-döntés, elfogadás, admin override, saját-elfogadás, értesítés-kiküldés, lezárás, kiigazítás): ki, mi, mikor, forrással. Bármely elfogadási/admin-döntési státuszváltás naplóbejegyzést kap forrással, időponttal és érintett entitással; az audit adat korlátlanul megőrizve.

## Context
Keresztmetszeti megbízhatóság: minden állapotváltás visszakereshető, korlátlan megőrzéssel.

## BDD Test
```gherkin
# language: hu
# Spec: belso-hozzaferes › Teljes audit napló (change: aok-osszesito-mvp)
@EPIC-belso-hozzaferes @STORY-belso-hozzaferes-audit-naplo
Jellemző: belso-hozzaferes
  Minden lényegi művelet naplózott (ki, mi, mikor, forrás), korlátlan megőrzéssel.

  Szabály: Teljes audit napló

    Forgatókönyv: Státuszváltás naplózása
      Adott egy elfogadási vagy admin-döntési státusszal rendelkező tétel
      Amikor a státusza megváltozik (token vagy admin forrásból)
      Akkor a rendszer naplóbejegyzést hoz létre a forrással, az időponttal és az érintett entitással
```

## Estimation
- **3-Points becslés (ideális óra):** O 6 / M 10 / P 20 | **Eβ 11**, σ 2,3
- **Becsült munkaóra:** 11 ó (≈ 1,8 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Kapcsolódás: minden más epic naplózandó eseményeket termel (`EPIC-excel-ingest`, `EPIC-targyfelelos-elfogadas`, `EPIC-idoszak-lezaras`, `EPIC-dij-szamitas`).
- Kockázat (GDPR): a korlátlan audit-megőrzés jogalapja (Open Question D); nyers adóazonosító kizárása a naplóból.
