# ÁOK Oktató- és tárgyfelelős összesítő — MVP

> Forrás: IT projektindítási igényfelmérő checklist v1.0 · BA: Csonka László · PM: Bilisics Bálint · Sponsor: IG/ÁOK – Biró László
> Frissítve a valós HR-Excelek elemzése és a 2026-07-01-i grillezés döntései alapján.

## Why

Az oktatói óra- és **díjelszámolás** ma manuális, papíralapú: a HR két Exceljéből (óranyilvántartás + szerződések) összeállított listát kinyomtatják, sorról sorra kézzel ellenőrzik és papíron aláírják, ami időszakonként **~600 oldalas nyomtatott paksaméta**. Ez lassú, hibalehetőségekkel teli, nehezen visszakövethető, és a jelenlegi szervert amúgy is le kell cserélni. A cél egy webes, adatbázis-alapú, auditálható folyamat, amely a megtartott órákból **oktatónkénti díjat** számol, tárgyfelelősi igazolással hitelesít, és a végén **egy áttekintő oldalt** állít elő — kevesebb manuális munka, kevesebb hiba, átláthatóbb és szűrhető adat. **Ha minden más elbukik, az összesítésnek (óra- és díjszámítás) korrektnek és megbízhatónak kell lennie.**

## What Changes

- **Egyszeri Excel-feltöltés + JOIN.** Laci (admin) együtt feltölti a 2 HR-Excelt: az **óranyilvántartást** (foglalkozás-soronkénti oktatási adat) és a **szerződés-listát** (szerződés-státusz + e-mail). A rendszer az **adóazonosítón** JOIN-olja, validálja, a hibás sorokat **karanténba**, az eldönthetetleneket **admin-döntésre** teszi.
- **Óra→díj számítás (a rendszer valódi terméke).** A díj soronként `tanóra × kategória-tétel`, ahol `tanóra = Órahossz/45`. Tételek/tanóra: előadás 21 000, szeminárium/gyakorlat 10 500, testnevelési/nyelvi 7 200. A díjkategória a `Kurzustípus` + egy **tárgy-szintű besoroló referenciatábla** (testnevelés/nyelvi kivételek). A **jogosultság** a szerződés-`státusz` × `Kurzus nyelve`: EÜ szolgálati jogviszony / MÁS (nem PTE) minden nyelven fizetett; minden más csak idegennyelvű órán.
- **„Az eldönthetetlent az ember dönti el."** Ahol egy korrektség-kritikus érték nem determinisztikus (díjkategória, státusz-ütközés, párosítatlan oktató, lezárt hónapba eső késői sor), a rendszer a sort az admin felületen **választható opcióval** jeleníti meg; sosem tippel csendben, és nem dob el mindent karanténba.
- **Token-alapú tárgyfelelős-elfogadás (a papíraláírás kiváltása).** A tárgyfelelős login nélküli token-linken **(tárgy × oktató) soronként** az **órákat** fogadja el (díjat nem lát). Az elfogadás végleges; visszavonni nem tud, csak az admin írhatja felül.
- **Elfogadás = kifizetési kapu.** Csak az **elfogadott** órák díja végleges; a nem elfogadott (lezáráskor „átcsúszott") órák díja **függő** tétel (határidő nélkül, auto-write-off nélkül, korosítva). Ha a tárgyfelelős nem reagál, az **admin override** felszabadíthatja. **Saját-elfogadás** (a tárgyfelelős maga az oktató) engedett, de audit-jelölt. **Lezárás után** a függő díj csak admin override-dal, **kiigazításként** a következő nyitott időszakban rendeződik (a lezárt időszak érintetlen).
- **Havi időszakok az óra dátuma szerint.** Egy „időszak" = egy naptári hónap, a sor `Óra kezdete`-je alapján. Egy feltöltés **több hónapot** is hozhat; a rendszer hónaponként particionál. Több nyitott időszak létezhet; az admin **egyenként, manuálisan** zár. Lezárt időszak **változtathatatlan**.
- **Áttekintő összesítő PDF.** Lezáráskor egy áttekintő PDF: aggregált óra- és **díjösszegek**, elfogadott vs. átcsúszott/függő kiemelve, és egy **visszakereső azonosító**, amivel a papírpéldányból a részletes webes táblázat visszakereshető.
- **Belső táblázat-nézet díjjal.** Az admin (edit) és a HR/vezetők (read) M365-tel belépve látják a táblázatot (oktató × tárgy × óra × díj), szűrhetik/aggregálhatják.
- **Naplózás / auditálhatóság, korlátlan megőrzés.** Minden lényegi művelet visszakövethető. A nyers adóazonosító titkosítva, minden nézetből/PDF/logból kizárva.
- **Historikus adatok megjelenítése.** Friss start + egyszeri DB→DB migráció **read-only pillanatképként** — a régi eredményekhez nem nyúlunk, nem számoljuk újra.

## Capabilities

### New Capabilities
- `excel-ingest`: A 2 HR-Excel feltöltése, adóazonosító-JOIN, időszak-levezetés az óra dátumából (több-hónapos feltöltés), validáció, karantén és eldönthetetlen sorok admin-döntésre.
- `dij-szamitas`: Foglalkozás-alapú, tanóra-egységű díjszámítás; kategória- és jogosultság-levezetés (státusz × nyelv); tie-break és párosítatlan/eldönthetetlen esetek kezelése; kifizetés az elfogadáshoz kötve.
- `targyfelelos-elfogadas`: Token-link, login nélküli token-nézet, (tárgy × oktató) soronkénti **óra**-elfogadás, admin override, kifizetési kapu, token-kontrollok.
- `idoszak-lezaras`: Több nyitott időszak, manuális lezárás átcsúszott-jelöléssel és függő díjjal, immutabilitás, áttekintő PDF visszakereső azonosítóval.
- `belso-hozzaferes`: M365 auth, admin/read szerepkör, díjas táblázat-nézet (szűrés/aggregáció), admin-feloldó felület az eldönthetetlen sorokra, audit napló.
- `historikus-migracio`: Egyszeri, read-only pillanatkép-migráció; historikus díj nem számolódik újra; fázisolható.

### Modified Capabilities
<!-- Nincs meglévő (archivált) spec; minden capability új. -->

## Out of Scope (MVP)

- **Nem foglalkozás-alapú díjtételek** adathiány miatt: hallgató-alapú (írásbeli/szóbeli kollokvium, szigorlat), hét-alapú (orvosi/egyéb szakmai praxisgyakorlat), szakdolgozat-konzulensi díj. A díjmotor bővíthetőre tervezve, de MVP-ben csak a foglalkozás-alapú elszámolás.
- Teljes M365 belépés az oktatóknak / tárgyfelelősöknek (ők nem login-userek).
- Oktatói/tárgyfelelős webnézet: az oktató statikus PDF/Excel mellékletet kap (saját óra + saját díj), a tárgyfelelős token-nézetet (csak óra).
- Erős személyazonosítás az elfogadáshoz (token-kattintás elég; lásd Risk).
- „Nem fogadom el" gomb és automatikus emlékeztetők (a hallgatás = átcsúszott).
- Gépi bérszámfejtési Excel-export (későbbi fázis; a táblázat szűrése kiszolgálja).
- Dashboard / statisztikák.
- Tárgyfelelősönkénti aláírható PDF-ek (csak összesítő).
- Régi Excel-formátumok migrációja (csak a meglévő DB-ből, pillanatképként).

## Open Questions (becslés-blokkolók és feltételezések)

| # | Kérdés / hiányzó információ | Felelős | Hatás |
|---|------------------------------|---------|-------|
| A | E-mail küldés mechanizmusa: M365 Graph sendMail? Central api? relay? | IG / infra | **Blokkolja a becslést** — a token-link kiküldés ezen áll; interfész mögé tervezve |
| B | A jelenlegi rendszer forrás-DB sémája (migrációhoz) | jelenlegi rendszer gazdája | A migráció enélkül feltételes; fázisolható |
| C | Support-modell: gazda élesítés után, SLA, ártárgyalás | IG + BA | Release utáni felelősség, szerződés |
| D | GDPR-jogalap a nyers adóazonosító **korlátlan** megőrzéséhez | adatvédelmi / jogi | A megőrzési döntés miatt élesebb; ha kötelező a törlés, utólag kell törlő-mechanizmus |
| E | Tárgyfelelős-forrás: a tárgyfelelős (személy + e-mail) kiolvasható-e az Excelből? | ügyfél (továbbítva) | **Feltételezzük, hogy igen**; enélkül az elfogadási kör nem pontos |
| F | Nyelvi Intézet szaktantárgy-listája (mely Tárgykódok 10 500, nem 7 200) | ügyfél / Nyelvi Intézet | A besoroló referenciatábla feltöltéséhez |

## Risk (tudatosan vállalva)

- **Token = a teljes „aláírás", és most pénzt kapuz.** Nincs erős azonosítás; továbbküldött e-maillel más is elfogadhat, ami kifizetést indíthat. Vállalva, mert **nem gyengébb, mint a kiváltott papíraláírás** volt, és a rendszer csak a bérszámfejtés inputját állítja elő (HR-kör előzi a fizetést). Kompenzáció: ≥128 bit token, csak hash tárolva, lezáráskor érvénytelenítve, elfogadás-metaadat naplózva.
- **HR-adatminőség.** A mintában ~11,8% oktatónak nincs szerződés-párja (nincs státusz → jogosultság nem eldönthető), ~23% sornak nincs e-mailje, és 64 személynél jogosult+nem-jogosult státusz keveredik. Élesben az ügyfél tisztít, de a rendszer felkészül: admin-döntés, „kézbesíthetetlen" lista, tie-break jelöléssel.
- **HR-késés:** korábban 1-1 hetet késett az Excel; az egyszeri feltöltés és a kemény határidő (07.27 freeze, 08.15 átadás) mellett ez a fő ütemkockázat. A folyamat nem függ több feltöltéstől; a lezárás átcsúszottal feloldható.

## Impact

- **Új rendszer**, zöldmezős fejlesztés. **Stack: NestJS + Angular + PostgreSQL** (Linux/konténer az IT Igazgatóság infráján; M365/Entra auth).
- **Integrációk:** Excel import (~65 000 sor), PDF export, M365 auth (belső kör), e-mail küldés (mechanizmus tisztázandó — A), egyszeri migráció a jelenlegi DB-ből (B).
- **Adat:** személyes, oktatói, szerződéses és tárgy-adatok; adóazonosító mint join-kulcs (titkosítva); GDPR szigorú; auditálhatóság must-have.
- **Skála:** szerveroldali ingest ~65 000 soros Excelre (1× feltöltés); token-nézet ~100 egyidejű, könnyű olvasás + státuszflip; nem 2000 login-user.
- **Határidő:** 2026.07.27 feature freeze + bugfix + must-have CR; 2026.08.15 tesztelt, átadható állapot.
