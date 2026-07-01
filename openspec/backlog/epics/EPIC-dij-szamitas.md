# Díjszámítás — tanóra-alapú díj, kategória, jogosultság · EPIC-dij-szamitas

## Description
A rendszer valódi terméke: a megtartott foglalkozásokból oktatónkénti díjat számol.
A díj soronként `tanóra × kategória-tétel`, ahol `tanóra = Órahossz / 45`. A kategóriát
a `Kurzustípus` + egy tárgy-szintű besoroló referenciatábla adja (testnevelési/nyelvi
kivételek), a jogosultságot a szerződés-státusz × kurzusnyelv szabály. Minden sor önálló,
díjazott alkalom (társtanításnál nincs osztás). A fizetendő díj csak az elfogadott
órákból áll; a nem elfogadott órák díja függő.

## Persona
- Admin (Laci) — feloldja az eldönthetetlen kategóriát/státuszt, kezeli a párosítatlan
  oktatót.
- Oktató — a soronkénti díj jogosultja (nem login-user; statikus PDF/Excel mellékletet kap).
- HR — a párosítatlan (státusz nélküli) oktatók forrás-hiányát tisztázza.

## E2E Scenario
Ha a rendszer a megtartott foglalkozás-sorokra tanóra-alapú, kategóriához és
jogosultsághoz kötött díjat számol, akkor korrekt, soronként visszavezethető oktatói
díjösszeg áll elő a bérszámfejtés inputjához, mérve a determinisztikusan (admin-döntés
nélkül) besorolt sorok arányával és a számítás korrektségével.

## Problem / Solution
P: A díjszámítás ma manuális; a kategória (testnevelési/nyelvi kivétel), a jogosultság
(státusz × nyelv), a társtanítás és a párosítatlan oktató mind hibalehetőség, és ha bármi
más elbukik, az összegzésnek akkor is korrektnek kell lennie.

S: Egy determinisztikus, bővíthetőre tervezett díjmotor (kategória → elszámolási alap →
tétel): tanóra-alapú számítás, referenciatáblából vezetett kategória, státusz×nyelv
jogosultság, soronkénti teljes díj társtanításnál, és az eldönthetetlen esetek
admin-döntésre — sosem csendes tippelés.

## Cross-cutting Concerns
- Üzleti-logika korrektség: `tanóra = Órahossz/45`, tételek (előadás 21 000,
  szeminárium/gyakorlat 10 500, testnevelési/nyelvi 7 200) — a rendszer legkritikusabb
  helyessége.
- Kategória-levezetés: karbantartott, tárgy-szintű referenciatáblából (Tárgykód), NEM
  oszlop-heurisztikából — a Nyelvi Intézeten a 7 200 és a 10 500 minden strukturált
  mezőben azonos (ADR 0003); egy jövőbeli fejlesztő ne egyszerűsítse vissza.
- Társtanítás: minden sor teljes díj, slot-szintű osztás tilos (ADR 0007) — ne nézze
  senki duplikációnak.
- Adminisztratív döntés: eldönthetetlen kategória / státusz-ütközés / párosítatlan oktató
  admin-döntésre, sosem csendben alapértelmezve.
- Kifizetési kapu: a fizetendő díj csak elfogadott órákból; a nem elfogadott függő (ADR 0004).
- Bővíthetőség: a díjmodell kategória → elszámolási alap → tétel rétegre tervezve, de
  MVP-ben csak foglalkozás-alapú.

## MVP and Out of Scope
MVP:
- Foglalkozás-alapú, tanóra-egységű soronkénti díjszámítás; oktatónkénti összegzés.
- Csak megtartott sorok (`Nemindul` = Hamis); társtanítás minden sor teljes díj.
- Kategória- és tétel-levezetés referenciatáblából (testnevelési/nyelvi kivétel).
- Jogosultság státusz × kurzusnyelv szerint; magyar óra 0 Ft a nem kivételes státuszoknál.
- Szerződés-státusz feloldása, dátum-alapú ütközésfeloldás, jogosult-preferencia + admin-jel.
- Párosítatlan oktató admin-döntésre, díj függőben, HR-hiánylista.
- Fizetendő = csak elfogadott órák; nem elfogadott = függő tétel.

Out of Scope:
- Nem foglalkozás-alapú díjtételek (hallgató-alapú, hét-alapú, szakdolgozat-konzulensi).
- A tárgyfelelősi elfogadás mechanikája (lásd `EPIC-targyfelelos-elfogadas`).
- Bérszámfejtési Excel-export.

## Success metrics
- A megtartott, feloldott sorok díja soronként visszavezethető (`tanóra × tétel`), 0
  számítási eltéréssel a validációs mintán.
- Eldönthetetlen kategória / státusz / párosítatlan 0 csendben alapértelmezve.
- Magyar nyelvű, nem kivételes státuszú órák díja 0 Ft.

## Risks and Dependencies
- Függőség: a besoroló referenciatábla + adatgazda (Nyelvi Intézet szaktantárgy-listája,
  Open Question F).
- Függőség: `EPIC-excel-ingest` `assignment` rekordjai (státusz, nyelv, kurzustípus).
- Függőség: `EPIC-targyfelelos-elfogadas` az elfogadási státuszért (kifizetési kapu).
- Kockázat: HR-adatminőség — státusz-ütközés (64 személy), párosítatlan oktató (~11,8%).

## High Level Acceptance Criteria
- 90 perces előadás-sor → 2 tanóra → 42 000 Ft; `Nemindul` ≠ Hamis sor nem számít.
- Társtanított slot minden oktatói sora teljes díjat kap, nincs slot-osztás.
- Testnevelési egység gyakorlata 7 200; Nyelvi Intézeti szaktantárgy a normál tétel;
  besorolatlan Tárgykód admin-döntésre.
- Közalkalmazott magyar órája 0 Ft, idegennyelvű fizetett; EÜ szolgálati jogviszony minden
  nyelven fizetett.
- Több szerződésnél a dátum dönt; feloldhatatlan ütközésnél jogosult státusz + admin-jel.
- Párosítatlan oktató admin-döntésre, díj függőben, HR-hiánylistán.
- Fizetendő csak elfogadott órákból; átcsúszott óra díja függő; admin override felszabadít.

## Estimation
- **Becsült munkaóra:** 62 ó (≈ 10,3 ideális nap)
- **Story Point:** 34

<!-- pipeline-only:start -->
## User story-k
- `STORY-dij-szamitas-foglalkozas-alapu-szamitas`
  Mint oktató, szeretném, hogy a díjam soronként `tanóra × tétel` alapon, minden megtartott
  alkalmamra teljes díjjal számolódjon, hogy a társtanított óráim is hiánytalanul
  elszámoljanak.
  - 90 perces előadás → 2 tanóra → 42 000 Ft; `Nemindul` ≠ Hamis sor kihagyva.
  - Társtanított (kurzus, időpont) minden oktatói sora teljes díj, slot-osztás nélkül.
- `STORY-dij-szamitas-dijkategoria-levezetes`
  Mint admin, szeretném a díjkategóriát karbantartott referenciatáblából levezetni a
  testnevelési/nyelvi kivételekkel, hogy a tétel helyes legyen ott is, ahol az oszlopok
  nem árulják el.
  - Testnevelési egység gyakorlata 7 200, nem 10 500.
  - Nyelvi Intézeti szaktantárgy a normál tétel (pl. 10 500), nem 7 200.
  - Besorolatlan Nyelvi Intézeti Tárgykód admin-döntésre, díj nem véglegesítve.
- `STORY-dij-szamitas-jogosultsag`
  Mint admin, szeretném a fizetési jogosultságot státusz × kurzusnyelv szerint eldönteni,
  hogy a munkaköri kötelezettségként tartott magyar órák ne fizetődjenek ki tévesen.
  - Közalkalmazott magyar órája 0 Ft; idegennyelvű órája fizetett.
  - EÜ szolgálati jogviszony minden nyelven fizetett.
- `STORY-dij-szamitas-statusz-feloldas`
  Mint admin, szeretném a szerződés-státuszt a hash-kulcson feloldani és az ütközéseket
  dátum, majd jogosult-preferencia + jelölés szerint rendezni, hogy a többes szerződések ne
  torzítsák a jogosultságot.
  - Több szerződésnél az időszakra érvényes dátumú státusz alkalmazódik.
  - Feloldhatatlan ütközésnél a jogosult státusz + admin-jelölés felülvizsgálatra.
- `STORY-dij-szamitas-parositatlan-oktato`
  Mint HR, szeretném a szerződés-párt nem találó oktatókat forrás-hiányként listázva látni,
  hogy pótolhassam a hiányzó státuszt.
  - Feloldhatatlan adóazonosító → admin-döntésre jelölve, díj függőben, HR-hiánylistán.
- `STORY-dij-szamitas-fizetendo-elfogadott`
  Mint admin, szeretném, hogy a fizetendő végösszeg csak az elfogadott órákat tartalmazza,
  a nem elfogadottakat pedig függőként, hogy a kifizetési kapu érvényesüljön.
  - Átcsúszott (nem elfogadott) sor díja függő, kimarad a fizetendőből.
  - Admin override elfogadja → a díj a fizetendőbe kerül.

## Forrás-spec hivatkozás

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| STORY-dij-szamitas-foglalkozas-alapu-szamitas | EPIC-dij-szamitas | Foglalkozás-alapú díjszámítás tanórában | 90 perces előadás díja; Meg nem tartott sor kihagyása; Társtanított foglalkozás — mindenki teljes díj |
| STORY-dij-szamitas-dijkategoria-levezetes | EPIC-dij-szamitas | Díjkategória és tétel levezetése | Testnevelési gyakorlat kedvezményes tétele; Nyelvi Intézeten belüli szaktantárgy; Eldönthetetlen kategória admin-döntésre |
| STORY-dij-szamitas-jogosultsag | EPIC-dij-szamitas | Jogosultság szerződéstípus és kurzusnyelv szerint | Közalkalmazott magyar órája nem fizetett; Közalkalmazott idegennyelvű órája fizetett; EÜ szolgálati jogviszony minden nyelven fizetett |
| STORY-dij-szamitas-statusz-feloldas | EPIC-dij-szamitas | Szerződés-státusz feloldása és ütközés kezelése | Több szerződés, dátum dönt; Jogosult és nem-jogosult státusz ütközése |
| STORY-dij-szamitas-parositatlan-oktato | EPIC-dij-szamitas | Párosítatlan oktató kezelése | Nincs szerződés-pár |
| STORY-dij-szamitas-fizetendo-elfogadott | EPIC-dij-szamitas | A fizetendő díj csak elfogadott órákból | Átcsúszott óra díja függő; Admin override felszabadítja a díjat |
<!-- pipeline-only:end -->
