# Tárgyfelelős-elfogadás — token-link, kifizetési kapu · EPIC-targyfelelos-elfogadas

## Description
A papíralapú aláírásos igazolást váltja ki: a tárgyfelelős login nélküli, kriptográfiailag
erős token-linken (tárgy × oktató) soronként fogadja el az oktatók óráit — díjat nem lát.
Az elfogadás végleges és a kifizetési kapu: csak az elfogadott órák díja fizetendő.
Ha a tárgyfelelős nem reagál, az admin override felszabadíthatja. A saját-elfogadás
(a tárgyfelelős egyben oktató) engedett, de audit-jelölt. Az elfogadás gyenge
azonosítású — a napló ezt egyértelműen jelöli.

## Persona
- Tárgyfelelős — token-linken, login nélkül, (tárgy × oktató) soronként fogad el órákat.
- Admin (Laci) — override-dal elfogad, kezeli a kézbesíthetetlen listát, indítja az
  értesítést.
- Oktató — az elfogadott órái kapuzzák a saját kifizetését.

## E2E Scenario
Ha a tárgyfelelős token-linken, login nélkül, oktatónként elfogadja a tárgyán tartott
órákat, akkor a papíraláírással egyenértékű, auditált kifizetési kapu áll elő az ÁOK
elszámolásához, mérve a token útján (admin override nélkül) rendezett elfogadási sorok
arányával.

## Problem / Solution
P: A tárgyfelelősi igazolás ma papíron, kézzel aláírva történik (~600 oldal / időszak),
lassan és nehezen visszakövethetően; a tárgyfelelősök nem login-userek.

S: Login nélküli token-link (tárgy × oktató) soronkénti óra-elfogadással, ami egyben a
kifizetési kaput is nyitja; admin override a nem reagáló tárgyfelelősre, saját-elfogadás
audit-jelöléssel, és a token gyenge azonosításának explicit naplózása.

## Cross-cutting Concerns
- Biztonság: ≥128 bit token-entrópia, csak hash tárolva, a nyers token csak a linkben;
  lezáráskor minden token érvénytelen.
- Identitás korlátja: a token nem erős személyazonosítás — továbbküldött e-maillel más is
  elfogadhat; tudatosan vállalt, mert nem gyengébb a kiváltott papíraláírásnál (Risk).
- Adatkezelés: a token-nézet MUST NOT tartalmazzon díjat (forintot) és nyers adóazonosítót
  (sem URL-ben, sem tartalomban).
- Kifizetési kapu: csak az elfogadott órák díja végleges (ADR 0004).
- Auditálhatóság: az elfogadás metaadata (időpont, forrás, token esetén IP/user-agent)
  naplózott; a saját-elfogadás külön jelölt és szűrhető.
- Kézbesíthetőség: e-mail nélküli tárgyfelelős a „kézbesíthetetlen" listára kerül,
  admin override-ra.
- Lokalizáció: a token-nézet és az e-mail magyar nyelvű.

## MVP and Out of Scope
MVP:
- Admin által indított értesítés: egyedi token-link `(időszak, tárgyfelelős)` páronként.
- Login nélküli token-nézet: tárgyanként csoportosítva, oktatónkénti óra-lista, soronkénti
  „Elfogadom".
- Elfogadás végleges (egyirányú); részleges elfogadás nem blokkolja a többi oktatót.
- Admin override az elfogadási státuszra (kifizetési kaput is nyit).
- Elfogadás mint kifizetési kapu + audit (token = gyenge azonosítás jelölve).
- Saját-elfogadás engedett, audit-jelölt, belső nézetben szűrhető.

Out of Scope:
- Erős személyazonosítás az elfogadáshoz (token-kattintás elég).
- „Nem fogadom el" gomb és automatikus emlékeztetők (a hallgatás = átcsúszott).
- Tárgyfelelősi webnézet login-nal.
- Az e-mail küldés konkrét mechanizmusa (Open Question A — interfész mögé tervezve).

## Success metrics
- A token-nézetben 0 forintösszeg és 0 nyers adóazonosító.
- Token-elfogadások 100%-a auditált, forrás = `token`, gyenge azonosításként jelölve.
- Saját-elfogadások 100%-a külön jelölt és szűrhető.

## Risks and Dependencies
- Függőség: e-mail küldés mechanizmusa (Open Question A) — blokkolja a token-kiküldést.
- Függőség: a tárgyfelelős e-mailje az Excelből olvasható (Open Question E).
- Függőség: `EPIC-idoszak-lezaras` a token-érvényesség / lezárás határáért.
- Kockázat: token = teljes aláírás és pénzt kapuz; továbbküldött linkkel más is elfogadhat
  (tudatosan vállalva).

## High Level Acceptance Criteria
- Az „Értesítések kiküldése" minden tárgyfelelősnek egyedi token-linkes e-mailt küld,
  naplózva; a token csak hash-elve tárolt.
- E-mail nélküli tárgyfelelős a kézbesíthetetlen listára kerül.
- A token-nézet login nélkül, tárgyanként/oktatónként órákat mutat, forint és adóazonosító
  nélkül; az elfogadás végleges, részleges elfogadás nem blokkol.
- Lezárt időszak tokenje érvénytelen, nem enged elfogadást.
- Admin override rögzíti a státuszt (forrás `admin`), a díj a fizetendőbe kerül.
- Token-elfogadás auditja jelzi a gyenge azonosítást; a saját-elfogadás külön jelölt.

## Estimation
- **Becsült munkaóra:** 51 ó (≈ 8,5 ideális nap)
- **Story Point:** 27

<!-- pipeline-only:start -->
## User story-k
- `STORY-targyfelelos-elfogadas-token-kikuldes`
  Mint admin, szeretném az érintett tárgyfelelősöknek egyedi token-linket kiküldeni, hogy
  login nélkül elfogadhassák az órákat, biztonságosan tárolt tokennel.
  - Az értesítés minden tárgyfelelősnek személyre szabott token-linkes e-mailt küld,
    naplózva.
  - A token csak hash-elve tárolódik, a nyers token nem visszafejthető.
  - E-mail nélküli tárgyfelelős a „kézbesíthetetlen" listára kerül, admin override-ra.
- `STORY-targyfelelos-elfogadas-token-nezet`
  Mint tárgyfelelős, szeretném login nélkül, oktatónként elfogadni a tárgyaimon tartott
  órákat, díj nélkül, hogy a papíraláírást kiváltsa egy egyszerű token-nézet.
  - Egy oktató órasorának elfogadása `(időszak, tárgy, oktató)` státuszát `elfogadott`-ra
    állítja, forrás `token`, lezárva további módosítás elől.
  - Részleges elfogadás nem blokkolja a többi oktatót (nem mindent-vagy-semmit).
  - A nézet csak órákat mutat, forintot nem.
  - Lezárt időszakhoz tartozó token érvénytelen, nem enged elfogadást.
- `STORY-targyfelelos-elfogadas-admin-override`
  Mint admin, szeretném bármely elfogadási státuszt kézzel beállítani, ha a tárgyfelelős
  nem reagál, hogy a kifizetés ne akadjon el.
  - Az admin manuális `elfogadott` státuszt állít, forrás `admin` + kilét naplózva, a díj a
    fizetendőbe kerül.
- `STORY-targyfelelos-elfogadas-kifizetesi-kapu`
  Mint auditor, szeretném az elfogadást kifizetési kapuként és auditáltan kezelni, hogy
  csak az elfogadott órák díja legyen végleges és a token gyenge azonosítása látszódjon.
  - Token-elfogadás auditja: forrás `token`, időpont + technikai metaadat, gyenge
    azonosításként jelölve.
- `STORY-targyfelelos-elfogadas-sajat-elfogadas`
  Mint auditor, szeretném a saját-elfogadást engedni, de külön jelölni és szűrhetővé tenni,
  hogy az összeférhetetlenség átlátható maradjon.
  - A tárgyfelelős saját óráinak elfogadása rögzül, saját-elfogadás jelzővel naplózva,
    belső nézetben szűrhetően.

## Forrás-spec hivatkozás

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| STORY-targyfelelos-elfogadas-token-kikuldes | EPIC-targyfelelos-elfogadas | Token-link kiküldése a tárgyfelelősnek | Értesítés kiküldése; Token csak hash-elve tárolódik; Hiányzó tárgyfelelős e-mail |
| STORY-targyfelelos-elfogadas-token-nezet | EPIC-targyfelelos-elfogadas | Token-nézet és (tárgy × oktató) soronkénti elfogadás | Oktatónkénti elfogadás egy tárgyon; Egy vitás oktató nem blokkolja a többit; Díj nem jelenik meg a tárgyfelelősnek; Lejárt/lezárt időszak tokenje |
| STORY-targyfelelos-elfogadas-admin-override | EPIC-targyfelelos-elfogadas | Admin override az elfogadási státuszra | Admin beállítja az elfogadást |
| STORY-targyfelelos-elfogadas-kifizetesi-kapu | EPIC-targyfelelos-elfogadas | Elfogadás mint kifizetési kapu és audit | Token-elfogadás auditja a kilét korlátjával |
| STORY-targyfelelos-elfogadas-sajat-elfogadas | EPIC-targyfelelos-elfogadas | Saját-elfogadás engedélyezése és jelölése | Tárgyfelelős elfogadja a saját óráit |
<!-- pipeline-only:end -->
