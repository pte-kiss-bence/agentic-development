# Excel-ingest — feltöltés, JOIN, particionálás, karantén · EPIC-excel-ingest

## Description
Az admin (Laci) egyszerre feltölti a 2 HR-Excelt: az óranyilvántartást
(foglalkozás-soronkénti oktatási adat) és a szerződés-listát (szerződés-státusz +
e-mail). A rendszer a személy adóazonosítóján JOIN-olja a két fájlt, a sorokat az
`Óra kezdete` hónapja szerint havi időszakokhoz rendeli (egy feltöltés több hónapot
is hozhat), soronként validál, a hibás sorokat karanténba, az eldönthetetleneket
admin-döntésre teszi. Az adóazonosító titkosítva tárolt, HMAC-hash kulcson illesztve,
minden nézetből kizárva.

## Persona
- Admin (Laci) — feltölti a 2 Excelt, rendezi a karantént és az admin-döntéseket.
- HR — a forrás-Excelek gazdája; a párosítatlan/hiányos sorok visszajelzést kapnak.
- Adatvédelmi felelős — az adóazonosító titkosított kezelését, kizárását felügyeli.

## E2E Scenario
Ha az admin feltölti a 2 HR-Excelt, akkor a rendszer az adóazonosító-hash kulcson
összekapcsolt, hónap szerint particionált, soronként validált `assignment`
rekordokat állít elő a díjszámítás számára az ÁOK adminisztrációjának, mérve a
feloldott (betöltött) sorok arányával a karanténos és admin-döntésre váró sorokhoz
képest.

## Problem / Solution
P: A két HR-Excel ma kézzel, papíron áll össze; az adóazonosító-JOIN, a hónap szerinti
szétválasztás és a hibás sorok kiszűrése manuális, hibalehetőségekkel teli munka egy
~65 000 soros adathalmazon.

S: Egy szerveroldali ingest a két fájlt determinisztikus HMAC-hash kulcson JOIN-olja,
hónaponként particionál, soronként validál, a hibás sort karanténba, az eldönthetetlent
admin-döntésre teszi — sosem tippel csendben, és a nyers adóazonosítót titkosítva,
minden nézetből kizárva kezeli.

## Cross-cutting Concerns
- Adatkezelés / GDPR: az adóazonosító nyers értéke titkosítva tárolt, JOIN/keresés csak
  determinisztikus HMAC-hash kulcson; nyers érték kizárva minden nézetből, exportból,
  PDF-ből, tokenből és naplóból (ADR 0001).
- Biztonság: a nyers adóazonosítóhoz csak az ingest/migrációs komponens fér hozzá,
  audit-naplózva.
- Teljesítmény / skála: ~65 000 soros Excel egyszeri, szerveroldali feldolgozása.
- Adatminőség: a mintában ~11,8% oktatónak nincs szerződés-párja, ~23% sornak nincs
  e-mailje — az ingest felkészül (karantén, admin-döntés, hiánylista), nem dob el mindent.
- Auditálhatóság: a lezárt hónapba eső késői sor és a döntése (kiigazítás/elvetés)
  naplózott.
- Lokalizáció: az admin felület és a karantén/hibalisták magyar nyelvűek.

## MVP and Out of Scope
MVP:
- A 2 HR-Excel együttes feltöltése és adóazonosító-hash JOIN → `assignment` rekordok.
- Hónap szerinti particionálás az `Óra kezdete` alapján; több-hónapos feltöltés.
- Soronkénti validáció, karantén hibakóddal, eldönthetetlen sorok admin-döntésre.
- Betöltött / karantén / admin-döntésre váró sorok látható számlálása.
- Időszakonként egyszeri, végleges feltöltés (nincs felülíró újrafeltöltés).
- Adóazonosító titkosított tárolása + HMAC-hash illesztés.

Out of Scope:
- Régi Excel-formátumok migrációja (lásd `EPIC-historikus-migracio`).
- Ismételt / inkrementális felülíró feltöltés ugyanarra a nyitott időszakra.
- A díjkategória és a jogosultság levezetése (lásd `EPIC-dij-szamitas`).

## Success metrics
- A feloldható sorok 100%-ából `assignment` rekord keletkezik (a hibás sorok karanténban).
- Nyers adóazonosító 0 nézetben / exportban / naplóban.
- Eldönthetetlen sor 0 csendben alapértelmezve — mind admin-döntésre jelölve.

## Risks and Dependencies
- Függőség: az óranyilvántartás és a szerződés-lista oszlop-sémája (kötelező mezők).
- Függőség: HMAC-kulcs kezelése (titkosítás/kulcstár).
- Kockázat (GDPR): a nyers adóazonosító korlátlan megőrzésének jogalapja (Open Question D).
- Kockázat: HR-adatminőség (párosítatlan oktató, hiányzó e-mail) → admin-döntés terhe.

## High Level Acceptance Criteria
- 2 érvényes Excel feltöltése után a feloldható sorok `assignment`-ként rögzülnek,
  szerződés-státusszal és e-maillel, adóazonosító-hash kulcson JOIN-olva.
- Két hónapba eső sorok az `Óra kezdete` hónapja szerint külön időszakhoz rendelődnek.
- Lezárt hónapba eső sor nem módosítja a lezárt időszakot; admin-döntésre kerül, auditáltan.
- Hibás sor `quarantine_row`-ként rögzül hibakóddal, `assignment` nélkül.
- Eldönthetetlen érték (pl. díjkategória) admin-döntésre jelölve, díj nem véglegesítve.
- Ugyanazon nyitott időszak felülíró újrafeltöltése elutasítva.
- Nyers adóazonosító sehol meg nem jelenik; az illesztés HMAC-hash-en történik.

## Estimation
- **Becsült munkaóra:** 73 ó (≈ 12,2 ideális nap)
- **Story Point:** 35

<!-- pipeline-only:start -->
## User story-k
- `STORY-excel-ingest-feltoltes-join`
  Mint admin, szeretném a 2 HR-Excelt együtt feltölteni és az adóazonosítón
  JOIN-oltatni, hogy a foglalkozás-sorokból szerződés-státusszal és e-maillel ellátott
  `assignment` rekordok álljanak elő.
  - 2 érvényes Excel feltöltésekor a feloldható sorok `assignment`-ként rögzülnek az
    adóazonosító-hash kulcson JOIN-olva.
- `STORY-excel-ingest-idoszak-particionalas`
  Mint admin, szeretném, hogy a feltöltött sorok az `Óra kezdete` hónapja szerint a
  megfelelő havi időszakhoz kerüljenek, hogy egy több-hónapos feltöltés is helyesen
  szétváljon.
  - Két hónapba eső sorok két külön havi időszakhoz rendelődnek.
  - Lezárt hónapba eső sor nem módosítja a lezárt időszakot, admin-döntésre kerül,
    auditáltan.
- `STORY-excel-ingest-validacio-karanten`
  Mint admin, szeretném a hibás sorokat karanténban, az eldönthetetleneket admin-döntésre
  látni számlálóval, hogy sosem tippel a rendszer csendben, és lássam a feldolgozás
  eredményét.
  - Hiányzó mező / feloldhatatlan JOIN-kulcs → `quarantine_row` hibakóddal, `assignment`
    nélkül.
  - Eldönthetetlen érték → betöltve, de admin-döntésre jelölve, díj nem véglegesítve.
  - A betöltött / karantén / admin-döntésre váró sorszámok láthatók, a listák elérhetők.
- `STORY-excel-ingest-vegleges-feltoltes`
  Mint admin, szeretném, hogy egy időszak feltöltése végleges legyen, hogy egy felülíró
  újrafeltöltés ne törölhesse a már rögzített adatot.
  - Egy már feltöltött nyitott időszak teljes felülíró újrafeltöltése elutasítva, jelezve,
    hogy az adat rögzített.
- `STORY-excel-ingest-adoazonosito-vedelem`
  Mint adatvédelmi felelős, szeretném az adóazonosítót titkosítva, HMAC-hash kulcson
  illesztve tudni, hogy a nyers érték soha ne szivárogjon.
  - Nyers adóazonosító 0 felületen / exportban / PDF-ben / naplóban.
  - Az illesztés a determinisztikus HMAC-hash kulcson történik, a nyers érték felfedése
    nélkül.

## Forrás-spec hivatkozás

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| STORY-excel-ingest-feltoltes-join | EPIC-excel-ingest | A két HR-Excel feltöltése és JOIN-ja | Sikeres feltöltés és JOIN |
| STORY-excel-ingest-idoszak-particionalas | EPIC-excel-ingest | Időszak levezetése az óra dátumából, több-hónapos feltöltés | Egy feltöltés több hónapra; Lezárt hónapba eső sor |
| STORY-excel-ingest-validacio-karanten | EPIC-excel-ingest | Soronkénti validáció, karantén és eldönthetetlen sorok | Hibás sor karanténba kerül; Eldönthetetlen sor admin-döntésre; Karantén és betöltés látható számlálása |
| STORY-excel-ingest-vegleges-feltoltes | EPIC-excel-ingest | Egyszeri, végleges feltöltés időszakonként | Ismételt felülíró feltöltés tiltása |
| STORY-excel-ingest-adoazonosito-vedelem | EPIC-excel-ingest | Adóazonosító védett kezelése feldolgozáskor | Adóazonosító nem szivárog; Illesztés a nyers érték kitétele nélkül |
<!-- pipeline-only:end -->
