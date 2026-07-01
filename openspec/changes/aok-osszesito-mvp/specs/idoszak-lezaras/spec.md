## ADDED Requirements

### Requirement: Havi időszakok, több nyitott lehet, admin zár egyenként
A rendszer SHALL az időszakot naptári hónapként kezelni, az `Óra kezdete` hónapja szerint. Mivel egy feltöltés több hónapot is érinthet, egyszerre **több nyitott/vázlat** időszak is létezhet; a rendszer SHALL lehetővé tenni, hogy az admin az időszakokat **egyenként, manuálisan** zárja le.

#### Scenario: Több nyitott időszak egy feltöltésből
- **WHEN** egy feltöltés két hónap adatát tölti be
- **THEN** mindkét hónaphoz létrejön/feltöltődik a nyitott időszak, és az admin külön-külön zárhatja őket

### Requirement: Manuális lezárás átcsúszott-jelöléssel és függő díjjal
A rendszer SHALL lehetővé tenni az admin általi manuális lezárást akkor is, ha vannak `nem_elfogadott` (tárgy × oktató) sorok; ezeket a lezáráskor `átcsúszott`-ként SHALL megjelölni, és a hozzájuk tartozó díjat **függő** tételként, a fizetendőtől elkülönítve SHALL kezelni. Lezárás előtt a rendszer SHALL blokkolóan figyelmeztetni, ha van rendezetlen karantén- vagy admin-döntésre váró sor. Lezáráskor a rendszer SHALL minden tokent érvényteleníteni.

#### Scenario: Lezárás nem-elfogadottal
- **WHEN** az admin lezár egy időszakot, amelyben vannak `nem_elfogadott` sorok
- **THEN** a rendszer lezárja az időszakot, a `nem_elfogadott` sorokat `átcsúszott`-ként jelöli, a díjukat függő tételként kezeli, és minden tokent érvénytelenít

#### Scenario: Figyelmeztetés rendezetlen sorokra
- **WHEN** az admin lezárást indít, miközben van karanténos vagy admin-döntésre váró sor
- **THEN** a rendszer blokkoló figyelmeztetést ad, és a lezáráshoz külön megerősítést kér

### Requirement: Lezárt időszak sérthetetlensége
A rendszer SHALL a lezárt időszakot változtathatatlanként (immutable) kezelni: lezárás után az időszak aggregátumai és a generált PDF MUST NOT változzanak. A lezárt hónapba eső, később érkező sor SHALL admin-döntésre kerüljön (kiigazítás vagy elvetés), a lezárt időszak érintetlenül hagyásával; a kiigazítás auditáltan, elkülönítve jelenik meg.

#### Scenario: Késői sor nem módosítja a lezárt időszakot
- **WHEN** egy későbbi feltöltésben egy már lezárt hónapba eső sor érkezik
- **THEN** a rendszer a lezárt időszakot nem módosítja, a sort admin-döntésre jelöli, és a döntést auditálja

### Requirement: Lezárás utáni függő díj rendezése kiigazítással
A rendszer SHALL lehetővé tenni, hogy egy lezárt időszakban átcsúszottként maradt **függő díj** később rendeződjön anélkül, hogy a lezárt időszakot módosítaná: a felszabadított díj egy **kiigazítás** tételként a **következő nyitott időszakban** SHALL megjelenjen, az eredeti `(időszak, tárgy, oktató)` tételre hivatkozva. Mivel lezáráskor minden token érvénytelen, a lezárás utáni rendezés SHALL kizárólag **admin override** útján történjen (új tokent a rendszer MUST NOT adjon ki). A lezárt időszak PDF-je és aggregátuma MUST NOT változzon.

#### Scenario: Utólag felszabadított függő díj kiigazításként
- **WHEN** egy lezárt időszak átcsúszott sorát az admin utólag override-dal elfogadja
- **THEN** a rendszer a lezárt időszakot érintetlenül hagyja, és a felszabadított díjat kiigazítás tételként a következő nyitott időszakban rögzíti, az eredeti tételre hivatkozva, auditáltan

#### Scenario: Lezárás után nincs token
- **WHEN** egy tárgyfelelős egy már lezárt időszak tételét szeretné elfogadni
- **THEN** a rendszer nem ad ki új tokent; a rendezés csak admin override-dal lehetséges

### Requirement: Áttekintő összesítő PDF visszakereső azonosítóval
A rendszer SHALL lezáráskor egy áttekintő PDF-et generálni az időszakra: aggregált számok (össz óra, össz **fizetendő díj**, elfogadott vs. átcsúszott sorok/összegek), az átcsúszottak (függő díj) kiemelése, az időszakra hivatkozva. A PDF SHALL tartalmazzon egy **visszakereső azonosítót** (időszak-referencia/GUID), amellyel a papírpéldányból a webes felület teljes, részletes táblázata visszakereshető. A PDF MUST NOT tartalmazzon nyers adóazonosítót, és forintot csak **aggregált** szinten.

#### Scenario: PDF generálása lezáráskor
- **WHEN** az időszak lezárul
- **THEN** a rendszer egyetlen áttekintő PDF-et állít elő az aggregált óra- és díjösszegekkel, az átcsúszott/függő tételek kiemelésével, egy visszakereső azonosítóval, nyers adóazonosító nélkül
