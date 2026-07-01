# ÁOK Oktató- és Tárgyfelelős Összesítő

Havi oktatói óra- és díjelszámolás: a HR két Exceljéből számolt, tárgyfelelősi igazolással hitelesített, auditált összesítő, amely a papíralapú aláírásos folyamatot váltja ki.

## Language

### Struktúra

**Időszak**:
Egy naptári hónap, a foglalkozás `Óra kezdete` dátuma szerint. Az elszámolás és a lezárás egysége.
_Avoid_: hónap (köznapi értelemben), periódus

**Tárgy**:
Egy oktatási tantárgy (Tárgykód azonosítja). Egy tárgyhoz több kurzus tartozik, és egy tárgyfelelős felel érte.
_Avoid_: subject/course keverve

**Kurzus**:
Egy tárgy egy konkrét meghirdetett csoportja (Kurzuskód). Pontosan egy tárgyhoz tartozik; itt zajlik a tényleges oktatás.
_Avoid_: tárgy

**Foglalkozás**:
Egy megtartott óraalkalom — az óranyilvántartás egy sora, saját kezdő/záró időponttal. Hossza változó (45–270 perc), NEM fix egység. **Minden sor egy önálló, díjazott oktatási alkalom:** ha egy időpontot többen tartanak (társtanítás), az oktatónként külön, teljes sor, és mindegyik a teljes díjat kapja — nincs slot-szintű osztás.
_Avoid_: óra, tanóra (ezek mást jelentenek)

**Tanóra**:
A díjszámítás 45 perces egysége: `tanóra = Órahossz / 45`. A díj alapegysége.
_Avoid_: óra, foglalkozás

**Oktató**:
A foglalkozást ténylegesen megtartó személy — a soronkénti `Oktató neve` / `Adóazonosító`; ő a díj jogosultja. Nem a kurzus teljes oktatói névsora (`Kurzushoz rendelt oktatók`), ami csak információ.
_Avoid_: tanár, előadó, kurzus-roster

**Tárgyfelelős**:
A tárgyért felelős személy, aki igazolja (elfogadja) az oktatók óráit az adott tárgyon. Nem feltétlenül azonos az oktatóval.
_Avoid_: felelős, koordinátor

### Díjazás

**Díjkategória**:
A foglalkozás díjbesorolása, amiből a tétel jön. A kurzustípusból + tárgy-szintű besorolásból (testnevelési/nyelvi kivétel) vezetjük le, nem pusztán a kurzustípusból.
_Avoid_: kurzustípus (az csak a nyers adat)

**Tétel**:
Egy tanórára jutó díj forintban, a díjkategória szerint (pl. előadás 21 000, gyakorlat/szeminárium 10 500, testnevelési/nyelvi 7 200).
_Avoid_: díj, ár

**Jogosultság**:
Eldönti, hogy egy foglalkozás fizetett-e, a szerződés-státusz × kurzusnyelv szabály alapján.

**Fizetendő díj**:
Az elfogadott órák véglegesített díja.

**Függő díj**:
A nem elfogadott (átcsúszott) órák visszatartott díja; nem része a fizetendőnek, amíg nem rendeződik. Határidő nélkül függő marad (nincs automatikus elengedés); bármikor rendezhető kiigazítással. A belső nézet korosítva (aging) mutatja.
_Avoid_: visszatartott (szinonima; a „függő" a kanonikus)

**Kiigazítás**:
Egy későbbi, nyitott időszakban könyvelt tétel, amely egy lezárt időszak elemére hivatkozik, anélkül hogy azt módosítaná. Így fizetődik ki utólag egy lezárás után felszabadított függő díj — a lezárt időszak befagyva marad. Lezárás után a rendezés csak admin override-dal történik (a tokenek már érvénytelenek).
_Avoid_: korrekció, módosítás (a lezártat nem módosítjuk)

### Folyamat és státuszok

**Elfogadás**:
A tárgyfelelős (vagy admin override) igazolja, hogy egy oktató a tárgyán a **foglalkozásait** ténylegesen megtartotta (tanórában mérve) — az egység a **(tárgy × oktató)** sor (a kurzusok összevonva; a kurzus- és foglalkozás-bontás csak lenyitható részlet). Ez a kifizetési kapu. Csak órát érint, díjat nem. Egy tárgynak pontosan egy tárgyfelelőse van.
_Avoid_: jóváhagyás, engedélyezés

**Átcsúszott**:
Egy elfogadási sor státusza, amit a lezárásig nem fogadtak el; a díja függő lesz.

**Admin override**:
Az admin manuális elfogadása, ha a tárgyfelelős nem tette meg. Az elfogadás egyik forrása.

**Admin-döntés (feloldás)**:
Az admin választása egy eldönthetetlen ADAT-kérdésben (díjkategória, státusz-ütközés, párosítatlan oktató, lezárt hónapba eső sor). Külön fogalom az elfogadástól: adatbesorolás, nem óra-igazolás.
_Avoid_: elfogadás (más fogalom!)

**Saját-elfogadás**:
Amikor a tárgyfelelős egyben oktató is a saját tárgyán, és a saját óráit fogadja el (a saját kifizetését kapuzva). Engedett, de az audit külön megjelöli és a belső nézetben szűrhető.

**Lezárás**:
Az admin véglegesíti egy időszakot: a nem elfogadott sorok átcsúszottá válnak (díjuk függő), a tokenek érvénytelenné válnak, elkészül az áttekintő PDF, és az időszak változtathatatlanná (immutable) válik.
_Avoid_: zárás, véglegesítés (általánosan)

**Karantén**:
A betölthetetlen (hibás vagy feloldhatatlan) sorok elkülönített helye, hibakóddal.

### Adat

**Adóazonosító**:
A személy azonosítója, a két Excel JOIN-kulcsa. Érzékeny; titkosítva tárolt, minden nézetből kizárt; HMAC-hash-en illesztünk.
_Avoid_: adószám (a szerződés-fájl így hívja, de ugyanaz), személyi szám

**Szerződés-státusz**:
A személy foglalkoztatási jogviszonyának típusa (pl. EÜ szolgálati jogviszony, közalkalmazott); a jogosultság bemenete.
_Avoid_: státusz (túl általános), szerződéstípus
