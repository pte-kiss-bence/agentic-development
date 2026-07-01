## ADDED Requirements

### Requirement: Token-link kiküldése a tárgyfelelősnek
A rendszer SHALL az admin által indított értesítéskor minden érintett tárgyfelelősnek egyedi, kriptográfiailag erős (≥128 bit entrópia) token-linket küldeni, amely egy `(időszak, tárgyfelelős)` párra érvényes. A token tárolva SHALL csak **hash** formában legyen; a nyers token csak a kiküldött linkben szerepelhet. A token érvényessége az időszak lezárásáig tart. A tárgyfelelős azonosítója és e-mail címe az óranyilvántartásból (Excel) származik (feltételezett forrás — lásd Open Question).

#### Scenario: Értesítés kiküldése
- **WHEN** az admin a karantén/admin-döntések rendezése után az „Értesítések kiküldése" műveletet indítja
- **THEN** a rendszer minden tárgyfelelősnek személyre szabott e-mailt küld egyedi token-linkkel, és a kiküldést naplózza

#### Scenario: Token csak hash-elve tárolódik
- **WHEN** a rendszer létrehozza a token-rekordot
- **THEN** csak a token hash-e kerül tárolásra, a nyers token nem visszafejthető az adatbázisból

#### Scenario: Hiányzó tárgyfelelős e-mail
- **WHEN** egy tárgyfelelőshöz nincs e-mail cím
- **THEN** a rendszer a tételt „kézbesíthetetlen" listára teszi az admin számára, és az elfogadás admin override útján oldható meg

### Requirement: Token-nézet és (tárgy × oktató) soronkénti elfogadás
A rendszer SHALL a token-linkre érkező tárgyfelelősnek **login nélkül** megjeleníteni a hozzá tartozó tárgyakat, tárgyanként csoportosítva, és **oktatónként** az adott tárgyon összesített **órákat** csak-olvasható listaként, oktatói soronkénti „Elfogadom" lehetőséggel. A nézet MUST NOT tartalmazzon díjazást (forintot) és nyers adóazonosítót (sem az URL-ben, sem a tartalomban). Az elfogadás SHALL végleges (egyirányú) legyen; a tárgyfelelős MUST NOT tudja visszavonni.

#### Scenario: Oktatónkénti elfogadás egy tárgyon
- **WHEN** a tárgyfelelős egy tárgyon belül egy oktató órasorát megnyitja és az „Elfogadom"-ra kattint
- **THEN** a rendszer az adott `(időszak, tárgy, oktató)` státuszát `elfogadott`-ra állítja, forrásként `token`-t rögzít, és a sort lezárja további tárgyfelelősi módosítás elől

#### Scenario: Egy vitás oktató nem blokkolja a többit
- **WHEN** egy tárgyon több oktató szerepel, és a tárgyfelelős csak néhányat fogad el
- **THEN** a rendszer az elfogadott oktatói sorokat elfogadottként rögzíti, a többit érintetlenül hagyja (nem mindent-vagy-semmit)

#### Scenario: Díj nem jelenik meg a tárgyfelelősnek
- **WHEN** a tárgyfelelős megnyitja a token-nézetet
- **THEN** a nézet csak órákat mutat, forintösszeget nem

#### Scenario: Lejárt/lezárt időszak tokenje
- **WHEN** a token egy már lezárt időszakhoz tartozik
- **THEN** a rendszer érvénytelennek jelzi a linket, és nem enged elfogadást

### Requirement: Admin override az elfogadási státuszra
A rendszer SHALL lehetővé tenni, hogy az admin bármely `(időszak, tárgy, oktató)` elfogadási státuszt kézzel beállítson, ha a tárgyfelelős nem reagált vagy nem elérhető. Az admin override SHALL a kifizetési kaput is felszabadítsa (az így elfogadott óra a fizetendő díjba kerül).

#### Scenario: Admin beállítja az elfogadást
- **WHEN** az admin egy oktatói sort manuálisan `elfogadott`-ra állít
- **THEN** a rendszer a státuszt rögzíti, forrásként `admin`-t és az admin kilétét naplózza, és a sor díja a fizetendő összegbe kerül

### Requirement: Elfogadás mint kifizetési kapu és audit
A rendszer SHALL az elfogadást (token vagy admin) kezelni a kifizetés kapujaként: csak az elfogadott órák díja végleges. A rendszer SHALL az elfogadás metaadatát (időpont, forrás, valamint token esetén technikai jelzők, pl. IP/user-agent) naplózni, és az auditban egyértelműen jelezni, hogy a token-elfogadás **nem** erős személyazonosítás.

#### Scenario: Token-elfogadás auditja a kilét korlátjával
- **WHEN** egy elfogadás token útján történik
- **THEN** a napló a művelet forrását `token`-ként, az időponttal és a technikai metaadattal rögzíti, és jelzi, hogy ez nem erős személyazonosítás

### Requirement: Saját-elfogadás engedélyezése és jelölése
A rendszer SHALL megengedni, hogy a tárgyfelelős a saját, oktatóként megtartott óráit is elfogadja (saját-elfogadás), de ezt SHALL az auditban külön megjelölni (self-approval jelző), és a belső táblázat-nézetben SHALL szűrhetővé tenni. A rendszer MUST NOT tiltsa a saját-elfogadást, de MUST NOT hagyja jelöletlenül.

#### Scenario: Tárgyfelelős elfogadja a saját óráit
- **WHEN** a tárgyfelelős egy olyan (tárgy × oktató) sort fogad el, ahol az oktató saját maga
- **THEN** a rendszer az elfogadást rögzíti, saját-elfogadás jelzővel naplózza, és a belső nézetben szűrhetővé teszi
