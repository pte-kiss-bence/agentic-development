# Adóazonosító titkosított kezelése és HMAC-illesztés · STORY-excel-ingest-adoazonosito-vedelem

Parent epic: `EPIC-excel-ingest`

## Description
Mint **adatvédelmi felelős**, szeretném **az adóazonosítót titkosítva, HMAC-hash kulcson illesztve tudni**, hogy **a nyers érték soha ne szivárogjon**.

A rendszer az adóazonosító nyers értékét titkosítva tárolja (megőrizve a jövőbeli igényekre), a JOIN/keresés determinisztikus HMAC-hash kulcson megy, és a nyers értéket minden nézetből, exportból, PDF-ből, tokenből és naplóból kizárja; hozzáférés csak az ingest/migrációs komponensnek, audit-naplózva.

## Context
GDPR-kritikus adatkezelés: az érzékeny nemzeti azonosító megőrizve, de kizárva mindenhonnan (ADR 0001).

## BDD Test
```gherkin
# language: hu
# Spec: excel-ingest › Adóazonosító védett kezelése feldolgozáskor (change: aok-osszesito-mvp)
@EPIC-excel-ingest @STORY-excel-ingest-adoazonosito-vedelem
Jellemző: excel-ingest
  Az adóazonosító titkosítva tárolt, HMAC-hash kulcson illesztve, minden nézetből kizárva.

  Szabály: Adóazonosító védett kezelése feldolgozáskor

    Forgatókönyv: Adóazonosító nem szivárog
      Adott egy feldolgozott adathalmaz, amely titkosított adóazonosítót tartalmaz
      Amikor a rendszer naplóz, vagy az admin megnyitja a karantén/eredmény nézetet
      Akkor a nyers adóazonosító egyetlen felületen, exportban, PDF-ben vagy naplóbejegyzésben sem jelenik meg

    Forgatókönyv: Illesztés a nyers érték kitétele nélkül
      Adott egy személy, akit a szerződés-adattal kell illeszteni
      Amikor a rendszer elvégzi az illesztést
      Akkor az a determinisztikus HMAC-hash kulcson történik
      És nem fedi fel a nyers adóazonosítót
```

## Estimation
- **3-Points becslés (ideális óra):** O 8 / M 12 / P 24 | **Eβ 13**, σ 2,7
- **Becsült munkaóra:** 13 ó (≈ 2,2 ideális nap)
- **Story Point:** 8

## Risks and Dependencies
- Függőség: titkosítási kulcs- és HMAC-kulcs-kezelés (kulcstár), `STORY-excel-ingest-feltoltes-join` (a hash-kulcs a JOIN-hoz).
- Kockázat (GDPR): a nyers adóazonosító korlátlan megőrzésének jogalapja (Open Question D) — ha felső korlát kötelező, utólag törlő/anonimizáló mechanizmus kell.
