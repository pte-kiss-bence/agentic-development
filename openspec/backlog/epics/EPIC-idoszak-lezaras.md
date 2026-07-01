# Időszak-lezárás — havi zárás, immutabilitás, PDF · EPIC-idoszak-lezaras

## Description
Az időszak egy naptári hónap, az `Óra kezdete` szerint. Egyszerre több nyitott időszak
lehet; az admin egyenként, manuálisan zár. Lezáráskor a nem elfogadott sorok átcsúszottá
válnak (díjuk függő), minden token érvénytelen, elkészül az áttekintő PDF, és az időszak
változtathatatlanná válik. A lezárt hónapba eső késői sor nem módosítja a lezárt
időszakot; a felszabadított függő díj kiigazításként a következő nyitott időszakban
rendeződik, admin override-dal.

## Persona
- Admin (Laci) — egyenként zárja az időszakokat, kezeli a késői sorokat és a kiigazítást.
- HR / bérszámfejtés — a lezáráskori áttekintő PDF-et és a fizetendő/függő bontást használja.
- Oktató — a felszabadított függő díját a következő időszak kiigazításaként kapja.

## E2E Scenario
Ha az admin manuálisan lezár egy havi időszakot, akkor egy változtathatatlan, auditált
állapot és egy visszakereshető áttekintő PDF áll elő az ÁOK elszámolásához, mérve a
lezárt időszakok immutabilitásának sértetlenségével (0 utólagos módosítás) és a
kiigazítással rendezett függő tételek követhetőségével.

## Problem / Solution
P: A havi elszámolás lezárása ma papíron véglegesül; nincs változtathatatlan pillanatkép,
a késői/hiányzó sorok és a HR-késés blokkolhatják a zárást, és a felszabaduló függő díj
utólagos kezelése nem visszakövethető.

S: Manuális, időszakonkénti lezárás átcsúszott-jelöléssel és függő díjjal (a HR-késés nem
blokkol), teljes immutabilitás lezárás után, késői sor admin-döntésre, és a függő díj
kiigazításként a következő nyitott időszakban — az eredeti tételre hivatkozva, auditáltan
(ADR 0006).

## Cross-cutting Concerns
- Immutabilitás: lezárás után az aggregátumok és a PDF MUST NOT változzanak (ADR 0002).
- Kifizetési kapu: a nem elfogadott sorok átcsúszottak, díjuk függő, a fizetendőtől
  elkülönítve.
- Biztonság: lezáráskor minden token érvénytelen; lezárás után csak admin override rendez
  (nincs új token).
- Auditálhatóság: a késői sor döntése és a kiigazítás (eredeti tételre hivatkozva)
  naplózott.
- Adatkezelés: a PDF MUST NOT tartalmazzon nyers adóazonosítót, forintot csak aggregált
  szinten.
- Visszakereshetőség: a PDF visszakereső azonosítót (időszak-referencia/GUID) hordoz a
  papírpéldány → webes részletes táblázat kötéshez.

## MVP and Out of Scope
MVP:
- Több nyitott/vázlat havi időszak; az admin egyenként, manuálisan zár.
- Manuális lezárás nem elfogadott sorokkal: átcsúszott-jelölés + függő díj; token-érvénytelenítés.
- Blokkoló figyelmeztetés rendezetlen karantén / admin-döntésre váró sorra.
- Lezárt időszak immutabilitása (aggregátum + PDF befagyva).
- Lezárás utáni függő díj rendezése kiigazításként a következő nyitott időszakban (admin
  override).
- Áttekintő összesítő PDF visszakereső azonosítóval.

Out of Scope:
- Tárgyfelelősönkénti aláírható PDF-ek (csak összesítő).
- Automatikus időszak-zárás / ütemezett zárás.
- A díjszámítás maga (lásd `EPIC-dij-szamitas`).

## Success metrics
- Lezárt időszakok 100%-a immutable (0 utólagos aggregátum-/PDF-változás).
- Minden lezáráskor pontosan 1 áttekintő PDF, visszakereső azonosítóval, nyers
  adóazonosító nélkül.
- Lezárás után 0 új token kiadva; a rendezés 100%-a admin override + kiigazítás.

## Risks and Dependencies
- Függőség: `EPIC-targyfelelos-elfogadas` elfogadási státuszaiért és a tokenekért.
- Függőség: `EPIC-dij-szamitas` a fizetendő/függő díjösszegekért.
- Függőség: PDF-generálás (aggregált óra/díj, kiemelt átcsúszott).
- Kockázat: HR-késés — a lezárás átcsúszottal feloldható, nem blokkolja a zárást.

## High Level Acceptance Criteria
- Egy több hónapos feltöltésből minden hónaphoz nyitott időszak jön létre, külön zárhatóan.
- Nem elfogadott sorokkal is lezárható: átcsúszott-jelölés, függő díj, minden token érvénytelen.
- Rendezetlen karantén / admin-döntés esetén blokkoló figyelmeztetés + külön megerősítés.
- Lezárt időszakba eső késői sor nem módosít; admin-döntésre kerül, auditáltan.
- Utólag felszabadított függő díj kiigazításként a következő nyitott időszakban, az eredeti
  tételre hivatkozva; lezárás után nincs új token.
- Lezáráskor 1 áttekintő PDF: aggregált óra/fizetendő díj, átcsúszott/függő kiemelve,
  visszakereső azonosító, nyers adóazonosító nélkül.

## Estimation
- **Becsült munkaóra:** 51 ó (≈ 8,5 ideális nap)
- **Story Point:** 28

<!-- pipeline-only:start -->
## User story-k
- `STORY-idoszak-lezaras-tobb-nyitott`
  Mint admin, szeretném egyszerre több nyitott havi időszakot kezelni és egyenként zárni,
  hogy egy több-hónapos feltöltés minden hónapja külön rendezhető legyen.
  - Két hónap adatát töltő feltöltésből mindkét hónaphoz nyitott időszak jön létre, külön
    zárhatóan.
- `STORY-idoszak-lezaras-manualis-lezaras`
  Mint admin, szeretném manuálisan lezárni az időszakot akkor is, ha vannak nem elfogadott
  sorok, hogy a HR-késés ne blokkolja a havi zárást.
  - Nem elfogadott sorokkal lezárva: átcsúszott-jelölés, függő díj, minden token érvénytelen.
  - Rendezetlen karantén / admin-döntés esetén blokkoló figyelmeztetés + külön megerősítés.
- `STORY-idoszak-lezaras-immutabilitas`
  Mint auditor, szeretném a lezárt időszakot változtathatatlannak, hogy a késői sor se
  módosíthassa az aggregátumot vagy a PDF-et.
  - Lezárt hónapba eső késői sor nem módosít; admin-döntésre kerül, auditáltan.
- `STORY-idoszak-lezaras-kiigazitas`
  Mint oktató, szeretném a lezárás után felszabadított függő díjamat a következő időszak
  kiigazításaként megkapni, hogy a lezárt hónap érintetlen maradjon.
  - Utólag override-dal elfogadott átcsúszott sor díja kiigazításként a következő nyitott
    időszakban, az eredeti tételre hivatkozva, auditáltan.
  - Lezárás után nincs új token; a rendezés csak admin override-dal.
- `STORY-idoszak-lezaras-osszesito-pdf`
  Mint HR, szeretném a lezáráskori áttekintő PDF-et visszakereső azonosítóval, hogy a
  papírpéldányból a webes részletes táblázat visszakereshető legyen.
  - Lezáráskor 1 áttekintő PDF: aggregált óra/díj, átcsúszott/függő kiemelve, visszakereső
    azonosítóval, nyers adóazonosító nélkül, forint csak aggregált szinten.

## Forrás-spec hivatkozás

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| STORY-idoszak-lezaras-tobb-nyitott | EPIC-idoszak-lezaras | Havi időszakok, több nyitott lehet, admin zár egyenként | Több nyitott időszak egy feltöltésből |
| STORY-idoszak-lezaras-manualis-lezaras | EPIC-idoszak-lezaras | Manuális lezárás átcsúszott-jelöléssel és függő díjjal | Lezárás nem-elfogadottal; Figyelmeztetés rendezetlen sorokra |
| STORY-idoszak-lezaras-immutabilitas | EPIC-idoszak-lezaras | Lezárt időszak sérthetetlensége | Késői sor nem módosítja a lezárt időszakot |
| STORY-idoszak-lezaras-kiigazitas | EPIC-idoszak-lezaras | Lezárás utáni függő díj rendezése kiigazítással | Utólag felszabadított függő díj kiigazításként; Lezárás után nincs token |
| STORY-idoszak-lezaras-osszesito-pdf | EPIC-idoszak-lezaras | Áttekintő összesítő PDF visszakereső azonosítóval | PDF generálása lezáráskor |
<!-- pipeline-only:end -->
