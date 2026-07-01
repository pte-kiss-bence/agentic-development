## ADDED Requirements

### Requirement: M365 alapú belső hozzáférés és szerepkörök
A rendszer SHALL a belső felhasználókat Entra ID (M365 / OIDC) útján hitelesíteni, és két szerepkört megkülönböztetni: `admin` (szerkesztés) és `reader` (csak olvasás). A `reader` MUST NOT végezhessen szerkesztő műveletet (feltöltés, admin-döntés, override, lezárás).

#### Scenario: Admin szerkeszthet
- **WHEN** egy `admin` szerepkörű felhasználó belép M365-tel
- **THEN** elérhetők számára a feltöltés, az admin-döntés/override és a lezárás műveletek

#### Scenario: Reader csak olvas
- **WHEN** egy `reader` szerepkörű felhasználó megnyit egy szerkesztő műveletet
- **THEN** a rendszer megtagadja a műveletet és nem módosít adatot

### Requirement: Táblázat-nézet díjjal, szűréssel és aggregációval
A rendszer SHALL a belső felhasználóknak megjeleníteni az időszak adatait táblázatként — oktató × tárgy × óra × **díj** —, szűréssel és aggregációval (pl. elfogadott/átcsúszott, függő díj, **saját-elfogadás**, szerződés-státusz, kurzusnyelv, tárgyfelelős szerint). A **függő díjat** a rendszer SHALL korosítva (aging) megjeleníteni, hogy a régóta függő tételek láthatók legyenek. A nézet MUST NOT tartalmazzon nyers adóazonosítót. (Gépi bérszámfejtési Excel-export MVP-n kívül, de a szűrés/aggregáció ezt a jövőbeli igényt kiszolgálja.)

#### Scenario: Szűrés átcsúszottakra
- **WHEN** a felhasználó az „átcsúszott" szűrőt alkalmazza
- **THEN** a táblázat csak a `nem_elfogadott` / `átcsúszott` (függő díjú) sorokat mutatja

#### Scenario: Függő díj korosítva
- **WHEN** a felhasználó a függő díjakat nézi
- **THEN** a rendszer korosítva (aging) mutatja, mennyi ideje függő az adott tétel

#### Scenario: Saját-elfogadás szűrése
- **WHEN** a felhasználó a „saját-elfogadás" szűrőt alkalmazza
- **THEN** a táblázat csak azokat a sorokat mutatja, ahol a tárgyfelelős a saját óráit fogadta el

#### Scenario: Aggregált díj oktatónként
- **WHEN** a felhasználó oktatónként aggregál
- **THEN** a rendszer oktatónként összesíti a fizetendő és a függő díjat, nyers adóazonosító nélkül

### Requirement: Eldönthetetlen sorok admin-feloldása a felületen
A rendszer SHALL az admin felületen megjeleníteni az admin-döntésre jelölt sorokat (pl. díjkategória, szerződés-státusz ütközés, párosítatlan oktató, lezárt hónapba eső késői sor), soronként **választható opcióval**, és a döntést eltárolni (auditáltan), hogy ismételt feldolgozáskor ne kelljen újra megkérdezni. A rendszer MUST NOT csendben alapértelmezni ezekben az esetekben.

#### Scenario: Admin feloldja a díjkategóriát
- **WHEN** egy sor eldönthetetlen díjkategória miatt admin-döntésre vár
- **THEN** az admin a felületen kiválasztja a helyes kategóriát, a rendszer a döntést auditáltan eltárolja, és a sor díja véglegesíthetővé válik

### Requirement: Teljes audit napló
A rendszer SHALL minden lényegi műveletet (ingest, admin-döntés, elfogadás, admin override, saját-elfogadás, értesítés-kiküldés, lezárás, kiigazítás) naplózni: ki, mi, mikor, forrással. Az audit adat SHALL korlátlanul megmaradjon és visszakövethető legyen.

#### Scenario: Státuszváltás naplózása
- **WHEN** bármely elfogadási vagy admin-döntési státusz megváltozik (token vagy admin forrásból)
- **THEN** a rendszer naplóbejegyzést hoz létre a forrással, az időponttal és az érintett entitással
