# Belső hozzáférés — M365 auth, táblázat, admin-feloldás, audit · EPIC-belso-hozzaferes

## Description
A belső kör (admin és HR/vezetők) M365-tel (Entra ID / OIDC) lép be. Két szerepkör van:
`admin` (szerkeszt: feltöltés, admin-döntés, override, lezárás) és `reader` (csak olvas).
A belső felhasználók táblázatként látják az időszak adatait (oktató × tárgy × óra × díj),
szűrhetik és aggregálhatják, a függő díjat korosítva. Az admin a felületen oldja fel az
eldönthetetlen sorokat választható opcióval. Minden lényegi művelet naplózódik (ki, mi,
mikor), és az audit adat korlátlanul megmarad.

## Persona
- Admin (Laci) — `admin` szerepkör; szerkeszt és feloldja az eldönthetetlen sorokat.
- HR / vezetők — `reader` szerepkör; olvas, szűr, aggregál.
- Adatvédelmi felelős — korlátlan audit-megőrzés jogalapja, adóazonosító-kizárás.

## E2E Scenario
Ha a belső felhasználók M365-tel, szerepkör szerint lépnek be, az adatot díjjal együtt,
szűrhető/aggregálható táblázatban látják, az admin a felületen oldja fel az eldönthetetlen
sorokat, és minden művelet auditált, akkor átláthatóvá és visszakövethetővé válik a
folyamat az ÁOK vezetése és adminisztrációja számára, mérve az auditált műveletek
lefedettségével és a szűréssel megválaszolt lekérdezések arányával.

## Problem / Solution
P: A mai folyamat nem visszakövethető, és a szűrhető, aggregálható adat helyett ~600
oldalas papírlista áll rendelkezésre; az eldönthetetlen sorok rendezésének sincs auditált
felülete.

S: Egy M365-alapú, szerepkörös belépés + díjas táblázat-nézet + felületi admin-feloldás +
teljes audit napló átláthatóvá és elszámoltathatóvá teszi a folyamatot: a vezetők szűrve
látják az adatot, az admin auditáltan old fel, minden státuszváltás visszakereshető — a
token-alapú elfogadás korlátját (gyenge azonosítás) a napló egyértelműen jelöli.

## Cross-cutting Concerns
- Biztonság: szerepkör-szeparáció szerver oldalon kikényszerítve — a `reader` nem végezhet
  szerkesztő műveletet, a megtagadás nem UI-szintű.
- Auditálhatóság: minden lényegi művelet (ingest, admin-döntés, elfogadás, override,
  saját-elfogadás, értesítés, lezárás, kiigazítás) naplózott, korlátlanul megőrizve.
- Adatkezelés / GDPR: nyers adóazonosító kizárása minden táblázat-nézetből és exportból;
  a korlátlan audit-megőrzés jogalapja tisztázandó (Open Question D).
- Adminisztratív döntés: az eldönthetetlen sorok admin-feloldása auditált, és a döntés
  eltárolódik, hogy ismételt feldolgozáskor ne kelljen újra megkérdezni; sosem csendes
  alapértelmezés.
- Naplózás: minden naplóbejegyzés hordozza a forrást, az időpontot és az érintett entitást.
- Identitás: a token-alapú elfogadás gyenge azonosítása a naplóban explicit jelölt.
- Lokalizáció: a felület és a táblázat-nézet magyar nyelvű.

## MVP and Out of Scope
MVP:
- Entra ID (M365 / OIDC) hitelesítés a belső körnek; MSAL + JWT-validáció (ADR 0005).
- `admin` és `reader` szerepkör; a `reader` nem végezhet szerkesztő műveletet.
- Táblázat-nézet díjjal (oktató × tárgy × óra × díj), szűréssel/aggregációval, függő díj
  korosítva, nyers adóazonosító nélkül.
- Eldönthetetlen sorok felületi admin-feloldása választható opcióval, auditáltan.
- Teljes audit napló minden lényegi műveletről, korlátlan megőrzéssel.

Out of Scope:
- Oktatók / tárgyfelelősök M365-belépése (ők nem login-userek).
- Gépi bérszámfejtési Excel-export (a szűrés/aggregáció a jövőbeli igényt előkészíti).
- Dashboard / statisztikák.
- Automatikus törlő / anonimizáló mechanizmus.

## Success metrics
- Reader-ből indított szerkesztő műveletek 100%-ban megtagadva.
- Lényegi műveletek 100%-a auditált.
- Nyers adóazonosító 0 táblázat-nézetben / exportban.
- Admin-döntésre jelölt sorok 100%-a felületen feloldható, auditáltan tárolva.

## Risks and Dependencies
- Függőség: Entra ID tenant + szerepkör-hozzárendelés.
- Függőség: naplózandó események forrásai (`EPIC-excel-ingest`,
  `EPIC-targyfelelos-elfogadas`, `EPIC-idoszak-lezaras`, `EPIC-dij-szamitas`).
- Kockázat (GDPR): korlátlan audit-megőrzés jogalapja.
- Kockázat: token-elfogadás gyenge azonosítása jelölendő.

## High Level Acceptance Criteria
- M365-tel belépő admin eléri a feltöltés / admin-döntés / override / lezárás műveleteket.
- Reader szerkesztő művelete szerver oldalon megtagadva, adatmódosítás nélkül.
- Táblázat-nézet díjjal szűrhető és aggregálható (átcsúszott, saját-elfogadás,
  oktatónkénti aggregáció), függő díj korosítva, nyers adóazonosító nélkül.
- Eldönthetetlen sor a felületen feloldható; a döntés auditáltan tárolódik, a díj
  véglegesíthetővé válik.
- Minden státuszváltás naplóbejegyzést kap forrással / időponttal / entitással.

## Estimation
- **Becsült munkaóra:** 46 ó (≈ 7,7 ideális nap)
- **Story Point:** 23

<!-- pipeline-only:start -->
## User story-k
- `STORY-belso-hozzaferes-m365-szerepkorok`
  Mint belső felhasználó, szeretném M365-tel belépni és a szerepkörömnek megfelelő jogot
  kapni, hogy csak az admin szerkeszthessen, a reader pedig biztonságosan csak olvasson.
  - Admin M365-belépés után eléri a feltöltés / admin-döntés / override / lezárás
    műveleteket.
  - Reader szerkesztő művelete szerver oldalon megtagadva, adatmódosítás nélkül.
- `STORY-belso-hozzaferes-tablazat-nezet`
  Mint HR/vezető, szeretném az időszak adatait díjjal, szűrhető/aggregálható táblázatban
  látni nyers adóazonosító nélkül, hogy gyorsan megválaszoljam a lekérdezéseket adatvédelmi
  kockázat nélkül.
  - Átcsúszott szűrő csak a `nem_elfogadott`/átcsúszott (függő díjú) sorokat mutatja.
  - A függő díj korosítva (aging) jelenik meg.
  - A saját-elfogadás szűrő csak a saját-elfogadott sorokat mutatja.
  - Oktatónkénti aggregáció összesíti a fizetendő és a függő díjat, adóazonosító nélkül.
- `STORY-belso-hozzaferes-admin-feloldas`
  Mint admin, szeretném az admin-döntésre jelölt sorokat a felületen, választható opcióval
  feloldani, hogy a döntés auditáltan eltárolódjon és ne kelljen újra megkérdezni.
  - Eldönthetetlen díjkategóriájú sornál az admin kiválasztja a kategóriát, a rendszer
    auditáltan tárolja, a sor díja véglegesíthetővé válik.
- `STORY-belso-hozzaferes-audit-naplo`
  Mint adatvédelmi felelős, szeretném minden lényegi művelet auditált naplóját, hogy a
  folyamat visszakövethető és elszámoltatható legyen.
  - Bármely elfogadási/admin-döntési státuszváltás naplóbejegyzést kap forrással,
    időponttal és entitással.

## Forrás-spec hivatkozás

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| STORY-belso-hozzaferes-m365-szerepkorok | EPIC-belso-hozzaferes | M365 alapú belső hozzáférés és szerepkörök | Admin szerkeszthet; Reader csak olvas |
| STORY-belso-hozzaferes-tablazat-nezet | EPIC-belso-hozzaferes | Táblázat-nézet díjjal, szűréssel és aggregációval | Szűrés átcsúszottakra; Függő díj korosítva; Saját-elfogadás szűrése; Aggregált díj oktatónként |
| STORY-belso-hozzaferes-admin-feloldas | EPIC-belso-hozzaferes | Eldönthetetlen sorok admin-feloldása a felületen | Admin feloldja a díjkategóriát |
| STORY-belso-hozzaferes-audit-naplo | EPIC-belso-hozzaferes | Teljes audit napló | Státuszváltás naplózása |
<!-- pipeline-only:end -->
