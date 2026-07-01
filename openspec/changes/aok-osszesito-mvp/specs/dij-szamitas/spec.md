## ADDED Requirements

### Requirement: Foglalkozás-alapú díjszámítás tanórában
A rendszer SHALL a foglalkozás-alapú kurzustípusokra (előadás, szeminárium, gyakorlat) a díjat soronként a **tanóra** egységre számítani, ahol `tanóra = Órahossz_perc / 45`, és a soronkénti díj `tanóra × a kategória Ft/tanóra tétele`. Az oktatóra jutó díj SHALL az időszakon belüli sorai díjának összege. A rendszer SHALL csak a ténylegesen megtartott sorokat (`Nemindul` = Hamis) beszámítani. **Minden sor egy önálló, díjazott oktatási alkalom;** a fizetett oktató a soronkénti `Oktató neve`/`Adóazonosító`. Ha ugyanazt a (kurzus, időpont) foglalkozást több oktató külön sorként tartja (társtanítás), a rendszer SHALL minden sort teljes díjjal számítani, és MUST NOT ossza el a díjat a slot oktatói között.

#### Scenario: 90 perces előadás díja
- **WHEN** egy megtartott előadás-sor `Órahossz`-a 90 perc
- **THEN** a rendszer 2 tanórát (90/45) számol, és a sor díja 2 × 21 000 = 42 000 Ft

#### Scenario: Meg nem tartott sor kihagyása
- **WHEN** egy sor `Nemindul` értéke nem „Hamis"
- **THEN** a rendszer a sort NEM számítja be a díjba

#### Scenario: Társtanított foglalkozás — mindenki teljes díj
- **WHEN** ugyanahhoz a (kurzus, időpont) foglalkozáshoz több oktató tartozik, külön-külön sorként
- **THEN** a rendszer minden sort önálló oktatási alkalomként, teljes díjjal számol, és nem oszt slot-szinten (pl. két oktató egy 45 perces előadásnál külön-külön 1-1 tanóra díját kapja)

### Requirement: Díjkategória és tétel levezetése
A rendszer SHALL a sor díjkategóriáját (Ft/tanóra tételét) a `Kurzustípus` alapján meghatározni — előadás 21 000, szeminárium 10 500, gyakorlat 10 500 —, azzal a felülírással, hogy a **testnevelési és nyelvi** kategória tétele 7 200. A testnevelési/nyelvi besorolás SHALL egy karbantartott, **tárgy-szintű besoroló referenciatáblából** (elsődlegesen `Szervezeti egység`, illetve a Nyelvi Intézeten belül `Tárgykód`-szintű kivétellista) származzon, NEM pusztán a `Kurzustípus`-ból. Ha egy sor kategóriája nem egyértelműen eldönthető, a rendszer SHALL a sort admin-döntésre jelölni, és MUST NOT csendben besorolni.

#### Scenario: Testnevelési gyakorlat kedvezményes tétele
- **WHEN** egy `Gyakorlat` sor `Szervezeti egység`-e a besoroló táblában testnevelési egységként szerepel (pl. „ÁOK Testnevelés- és Mozgásközpont")
- **THEN** a rendszer a tételt 7 200 Ft/tanórára állítja, nem 10 500-ra

#### Scenario: Nyelvi Intézeten belüli szaktantárgy
- **WHEN** egy sor a Nyelvi Intézethez tartozik, de a `Tárgykód`-ja a kivétellistán szaktantárgyként szerepel
- **THEN** a rendszer a normál kurzustípus-tételt alkalmazza (pl. 10 500), nem a 7 200-as nyelvi tételt

#### Scenario: Eldönthetetlen kategória admin-döntésre
- **WHEN** egy Nyelvi Intézeti sor `Tárgykód`-ja nincs a besoroló táblában
- **THEN** a rendszer a sort admin-döntésre jelöli a felületen (választható kategória), és a díjat addig nem véglegesíti

### Requirement: Jogosultság szerződéstípus és kurzusnyelv szerint
A rendszer SHALL a díjra való jogosultságot a személy szerződés-`státusz`-a és a sor `Kurzus nyelve` alapján eldönteni. `EÜ szolgálati jogviszony` vagy `MÁS (nem PTE) alkalmazott` státusz esetén minden nyelvű óra SHALL jogosult legyen. Minden más státusz esetén csak az idegennyelvű (nem magyar) óra SHALL jogosult; a magyar nyelvű óra díja 0 Ft.

#### Scenario: Közalkalmazott magyar órája nem fizetett
- **WHEN** egy `ÁOK közalkalmazott` státuszú oktató sorának `Kurzus nyelve` = magyar
- **THEN** a rendszer a sor díját 0 Ft-ra állítja (munkaköri kötelezettség)

#### Scenario: Közalkalmazott idegennyelvű órája fizetett
- **WHEN** ugyanezen oktató sorának `Kurzus nyelve` = angol vagy német
- **THEN** a rendszer a sort jogosultként kezeli és kiszámolja a díjat

#### Scenario: EÜ szolgálati jogviszony minden nyelven fizetett
- **WHEN** egy `EÜ szolgálati jogviszony` státuszú oktató magyar nyelvű órát tart
- **THEN** a rendszer a sort jogosultként kezeli (a magyar óráért is jár díj)

### Requirement: Szerződés-státusz feloldása és ütközés kezelése
A rendszer SHALL a személy szerződés-státuszát a JOIN-kulcson (adóazonosító-hash) feloldani. Ha egy személyhez több, eltérő státuszú szerződés tartozik, a rendszer SHALL az időszakra érvényes dátumú szerződés státuszát venni; ha így is holtverseny marad egy jogosult és egy nem-jogosult státusz között, a rendszer SHALL a jogosult státuszt alkalmazni, de a sort admin-jelöléssel megjelölni.

#### Scenario: Több szerződés, dátum dönt
- **WHEN** egy személynek két szerződése van eltérő státusszal, és csak az egyik érvényes az időszak dátumára
- **THEN** a rendszer az időszakra érvényes szerződés státuszát alkalmazza

#### Scenario: Jogosult és nem-jogosult státusz ütközése
- **WHEN** dátum alapján sem oldható fel az ütközés egy jogosult (pl. EÜ szolgálati jogviszony) és egy nem-jogosult (pl. ÁOK közalkalmazott) státusz között
- **THEN** a rendszer a jogosult státuszt alkalmazza, és a sort admin-jelöléssel jelöli felülvizsgálatra

### Requirement: Párosítatlan oktató kezelése
A rendszer SHALL a szerződés-fájlban párt nem találó (státusz nélküli) oktató sorait admin-döntésre jelölni, a hozzájuk tartozó díjat **függőben** tartani, és MUST NOT alapértelmezetten sem fizetettnek, sem nem-fizetettnek besorolni. Ezeket a rendszer SHALL a HR felé forrás-hiányként listázni.

#### Scenario: Nincs szerződés-pár
- **WHEN** egy tanító oktató adóazonosítója nem oldható fel a szerződés-fájlból
- **THEN** a rendszer a sort admin-döntésre jelöli, a díjat függőben tartja, és felveszi a HR-nek szánt hiánylistába

### Requirement: A fizetendő díj csak elfogadott órákból
A rendszer SHALL a fizetendő (végleges) díjba csak az **elfogadott** órák díját beszámítani (elfogadás token vagy admin override útján). A **nem elfogadott** (lezáráskor „átcsúszott") órák díja SHALL **függő** tételként, a fizetendőtől elkülönítve jelenjen meg. A függő díj SHALL határidő nélkül függő maradjon (nincs automatikus elengedés); lezárás utáni rendezése kiigazítással történik (lásd `idoszak-lezaras`). A rendszer SHALL a díjmodellt bővíthetőre tervezni (kategória → elszámolási alap → tétel), de MVP-ben csak a foglalkozás-alapú elszámolást SHALL megvalósítani.

#### Scenario: Átcsúszott óra díja függő
- **WHEN** egy oktató (tárgy × oktató) sora az időszak lezárásakor nem elfogadott
- **THEN** a rendszer a sor díját függő tételként tartja, és nem számítja a fizetendő végösszegbe

#### Scenario: Admin override felszabadítja a díjat
- **WHEN** az admin egy korábban nem elfogadott sort override-dal elfogad
- **THEN** a rendszer a sor díját a fizetendő végösszegbe sorolja
