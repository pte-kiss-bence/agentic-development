# A díjkategóriát karbantartott, tárgy-szintű referenciatábla adja

A foglalkozás díjkategóriáját (amiből a tétel jön) egy **karbantartott, tárgy-szintű (Tárgykód) besoroló referenciatábla** határozza meg, nem az Excel oszlopaiból levezetett heurisztika. A default a `Kurzustípus`, felülírva a testnevelési/nyelvi kivételekkel — de ezt **nem** lehet pusztán oszlopokból kiszámolni.

## Considered Options

- *Oszlop-heurisztika* (`Kurzustípus` + `Szervezeti egység` szöveg szerint) — **elvetve**. A Nyelvi Intézeten belül a 7 200-as nyelvi óra és a 10 500-as szaktantárgy **minden strukturált mezőben azonos** (Kurzustípus, Szervezeti egység, nyelv); csak a Tárgykód/Tárgynév különbözik. Emberi döntés kell tárgyanként.

## Consequences

- Kell egy karbantartott besoroló tábla + adatgazda (Nyelvi Intézet szaktantárgy-listája — Open Question F). Az eldönthetetlen tárgyakat az admin oldja fel a felületen. Egy jövőbeli fejlesztő ne „egyszerűsítse" vissza oszlop-levezetéssé — a Nyelvi Intézeten elhasal.
