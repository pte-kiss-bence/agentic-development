# Design — ÁOK Oktató- és tárgyfelelős összesítő (MVP)

> A motivációhoz lásd: [proposal.md](proposal.md). Ez a dokumentum a **hogyan**-t rögzíti.
> Frissítve a valós HR-Excelek elemzése és a 2026-07-01-i grillezés döntései alapján. Ahol a korábbi feltételezések tévesnek bizonyultak, a döntés melletti jegyzet jelzi.

## Context

Zöldmezős webalkalmazás, amely a ma papíralapú oktatói óra- **és díjelszámolást** váltja ki. A folyamat gazdája egyetlen admin (Laci); a tárgyfelelősök login nélkül, token-link útján fogadják el az órákat; a belső kör (HR/vezetők) M365-tel olvas. Időszak = naptári hónap az óra dátuma szerint. Kemény határidő: 2026.07.27 feature freeze, 2026.08.15 átadás.

Kulcs-kényszerek:
- **A rendszer valódi terméke a díjszámítás** — óra→forint, oktatónként. Ez a korrektség-kritikus mag.
- **GDPR „szigorú"** — a join-kulcs az **adóazonosító** (érzékeny nemzeti azonosító).
- **Auditálhatóság must-have**, korlátlan megőrzéssel.
- **Skála**: ~65 000 soros Excel egyszeri feldolgozása; token-nézet ~100 egyidejű.

## Adatforrások (valós Excelek)

- **Óranyilvántartás** (~65 000 sor, grain = egy megtartott **foglalkozás** soronként, saját `Óra kezdete`/`vége` dátummal). Kulcsmezők: `Órahossz percben` (45–270), `Tanóra (45 perces egység)` = `Órahossz/45`, `Adóazonosító` (JOIN-kulcs), `Oktató neve`, `Kurzustípus` (Előadás/Szeminárium/Gyakorlat), `Kurzus nyelve` (magyar/angol/német), `Szervezeti egység`, `Tárgykód`, `Nemindul` (Hamis=megtartott).
- **Szerződések** (személyenként több sor is): `adószám` (= óranyilvántartás `Adóazonosító`), `státusz` (6-féle szerződéstípus), `Email`, `Intézet`.
- **JOIN**: óranyilvántartás.`Adóazonosító` = szerződés.`adószám` (személy → státusz + e-mail). *(Eltérés az eredeti tervtől: a korábbi „adóazonosító + tárgyazonosító" JOIN téves volt — a tárgy adata teljes egészében az óranyilvántartásban van.)*

## Goals / Non-Goals

**Goals:**
- Egyszeri Excel-ingest adóazonosító-JOIN-nal, időszak-levezetéssel az óra dátumából, validációval, karanténnal és admin-döntéssel.
- **Óra→díj számítás** tanóra-egységen, kategória- és jogosultság-levezetéssel; kifizetés az elfogadáshoz kötve.
- Token-alapú, login nélküli, (tárgy × oktató) soronkénti **óra**-elfogadás + admin override.
- Több nyitott havi időszak, manuális lezárás átcsúszott/függő-jelöléssel + áttekintő PDF visszakereső azonosítóval.
- M365-auth a belső körre, díjas táblázat-nézet, admin-feloldó felület, teljes audit; adóazonosító védett kezelése.
- Egyszeri, read-only pillanatkép-migráció (nincs újraszámolás).

**Non-Goals:** lásd proposal „Out of Scope".

## Decisions

### D1 — Technológiai stack: NestJS + Angular + PostgreSQL
Node 22 LTS + **NestJS** backend + **Angular** SPA + **PostgreSQL** + Prisma/TypeORM; M365/Entra auth (MSAL a frontenden, JWT-validáció a backenden); e-mail interfész mögött (Graph sendMail vagy relay adapter); ExcelJS streamelt olvasás a 65k ingesthez; PDF pdfmake/hasonló.
**Miért:** a csapat NestJS/Angular-t használ; kemény határidőnél a meglévő tudásra építünk. *(Eltérés az eredeti tervtől: a korábbi .NET/SQL Server ajánlás elvetve — nem a csapat stackje; a „forrás-DB valószínűleg SQL Server" érv nem indokol .NET-et, az egyszeri ETL bármely stackből olvashat SQL Serverből.)* Linux/konténer az IT Igazgatóság infráján. DB-motor: Postgres (ha nincs IG-kikötés).

### D2 — Auth: két külön bizalmi sík
- **Belső kör:** Entra ID / OIDC (M365). Szerepkörök: `admin` (edit) és `reader` (HR/vezetők), DB-ben tárolt hozzárendeléssel.
- **Tárgyfelelős:** nincs auth; kriptográfiailag erős (≥128 bit), kitalálhatatlan **token** azonosít egy `(időszak, tárgyfelelős)` párt. A token a teljes „aláírás".

### D3 — Díjszámítás (a mag)
Soronkénti díj a foglalkozás-alapú kurzustípusokra:
- **tanóra = `Órahossz` / 45** (a `Tanóra` oszlop is `=H/45`). 1 foglalkozás = 90 perc = 2 tanóra.
- **díj(sor) = jogosult ? tanóra × tétel(kategória) : 0**, csak `Nemindul`=Hamis sorokra, oktatónként összegezve.
- **Tétel/tanóra:** Előadás 21 000 · Szeminárium 10 500 · Gyakorlat 10 500 · **testnevelési/nyelvi 7 200**.

### D4 — Díjkategória levezetése (referenciatáblával)
A kategória NEM pusztán a `Kurzustípus`-ból jön. Default: `Kurzustípus`. Felülírás egy **karbantartott, tárgy-szintű besoroló referenciatáblából**:
- `Szervezeti egység` = testnevelési egység (pl. „ÁOK Testnevelés- és Mozgásközpont") → 7 200.
- Nyelvi Intézet (pl. „Egészségügyi Nyelvi és Kommunikációs Intézet") → default 7 200, **kivéve** a `Tárgykód`-listán szereplő **szaktantárgy** → normál tétel. *(A strukturált oszlopok nem választják el a nyelvi órát a szaktantárgytól — egy „Medical Hungarian" és egy „Anatomical Terminology" sor minden strukturált mezőben azonos; csak a `Tárgykód`/`Tárgynév` különbözik. Ezért kell tárgy-szintű besorolás.)*
- Ahol a kategória nem eldönthető → **admin-döntésre** jelölve (nem tippel).

### D5 — Jogosultság (szerződéstípus × kurzusnyelv)
- `EÜ szolgálati jogviszony` **vagy** `MÁS (nem PTE) alkalmazott` → minden nyelvű óra fizetett.
- egyéb státusz (ÁOK/GYTK közalkalmazott stb.) → csak **idegennyelvű** (angol/német) fizetett; magyar = 0 Ft (munkaköri kötelezettség).
- **Több szerződés / ütközés:** az időszakra érvényes dátumú szerződés státusza dönt; ha jogosult és nem-jogosult státusz így is ütközik, a **jogosult** nyer, de a sor **admin-jelöléssel** megjelölve. *(A mintában 64 személynél tényleg keveredik — az ügyfél szerint élesben tiszta lesz, de a felület engedi a jelölést.)*
- **Párosítatlan oktató** (nincs szerződés → nincs státusz): jogosultság nem eldönthető → **admin-döntés**, díj **függőben**, HR-hiánylistára.

### D6 — Elfogadás és kifizetési kapu
- Elfogadás egysége a **(tárgy × oktató) sor**, tárgyanként csoportosítva; a tárgyfelelős **csak a foglalkozásokat** igazolja (tanórában), díjat nem lát. Egy tárgynak egy tárgyfelelőse van.
- **Csak elfogadott óra fizet.** Nem elfogadott = lezáráskor „átcsúszott" → díj **függő**, a fizetendőtől elkülönítve; a függő díj határidő nélkül függő marad (nincs auto-write-off), a belső nézet korosítva mutatja.
- Elfogadás forrása **token** vagy **admin override**; ha a tárgyfelelős nem reagál/nem elérhető, az admin elfogadhat (felszabadítja a díjat).
- **Saját-elfogadás** (a tárgyfelelős maga az oktató) engedett, de audit-jelölt és a belső nézetben szűrhető.
- **Lezárás után** a függő díj rendezése kizárólag **admin override**-dal történik (token nincs), és **kiigazítás** tételként a következő nyitott időszakban fizetődik ki (lásd D8).

### D7 — Token-mechanika
- Token-rekord: `(időszak_id, tárgyfelelős_id, token_hash, kibocsátva, érvénytelenítve)`. Nyers token csak a linkben; tárolva csak hash.
- Érvényesség az időszak lezárásáig; lezáráskor minden token érvénytelen.
- Az elfogadás metaadata (időpont, forrás, token esetén IP/user-agent) naplózva; az audit jelzi, hogy token ≠ erős azonosítás.
- **Adóazonosító és forint SOHA nem szerepel a tokenben, URL-ben, a tárgyfelelős-nézeten, a logban.**

### D8 — Időszak-modell (az óra dátuma alapján)
- Minden sor a saját `Óra kezdete` **hónapja** szerint kerül időszakhoz. Egy feltöltés több hónapot is hozhat → hónaponkénti particionálás.
- **Több nyitott/vázlat időszak** létezhet; az admin **egyenként, manuálisan** zár. *(Eltérés az eredeti tervtől: a korábbi „egyszerre egy nyitott" + „egyszeri feltöltés per időszak" szabály nem tartható; az adat több hónapba eshet.)*
- **Lezárt időszak immutable**; késői, lezárt hónapba eső sor → admin-döntés (**kiigazítás** vagy elvetés), a lezárt aggregátum/PDF érintése nélkül. A **kiigazítás** egy későbbi, nyitott időszakban könyvelt tétel, amely a lezárt időszak elemére hivatkozik — így fizetődik ki utólag a felszabadított függő díj is.

### D9 — Adóazonosító védelme (GDPR)
- **Tárolás titkosítva, a nyers érték megőrizve** (későbbi igényre; ma is az Excelben van). *(A „hash-only, nem tárolunk nyerset" alternatíva megfontolva, de elvetve.)*
- Join/keresés determinisztikus **HMAC-hash** kulcson.
- Nyers adóazonosító **kizárva** minden nézetből, PDF-ből, exportból, tokenből, naplóból; hozzáférés csak ingest/migráció, audit-naplózva.
- A korlátlan megőrzés **élesíti az Open Question D-t** (jogalap).

### D10 — Human-in-the-loop (keresztmetsző elv)
Ahol az ingest/díjszámítás egy korrektség-kritikus értéket nem tud determinisztikusan eldönteni (díjkategória, státusz-ütközés, párosítatlan oktató, lezárt hónapba eső sor), a rendszer a sort az **admin felületen választható opcióval** jeleníti meg, a döntést **auditáltan** eltárolja (újrafeldolgozáskor nem kérdez újra), és **sosem** defaultol csendben vagy dob el mindent karanténba.

### D11 — Kimenetek
- **Belső táblázat** (admin + reader): oktató × tárgy × óra × **díj**, szűrhető/aggregálható.
- **Áttekintő PDF** (lezáráskor): aggregált óra/díj, elfogadott vs. átcsúszott/függő, **visszakereső azonosító** a részletes táblázathoz. Forint csak aggregáltan, adóazonosító nélkül.
- **Oktatói e-mail**: saját óra + saját díj (statikus melléklet).
- **Tárgyfelelős e-mail + token**: csak óra.
- **Bérszámfejtési Excel-export: MVP-n kívül** (a táblázat szűrése előkészíti).

### D12 — Adatmodell magas szinten *(részletes oszlopstruktúra: külön)*
- `person` (person_id, név, `adoazonosito_hash` HMAC, titkosított adóazonosító, e-mail, szerződés-státusz(ok))
- `subject` (subject_id, tárgykód, megnevezés, szervezeti egység, besorolt díjkategória-felülírás)
- `period` (period_id, hónap, státusz `nyitott|lezárt`, lezárás_időpont, referencia-azonosító) — **több nyitott lehet**
- `assignment` (period_id, subject_id, oktató person_id, kurzustípus, nyelv, órahossz→tanóra, számított díj, jogosultság, díj-státusz `fizetendő|függő`)
- `acceptance_status` (period_id, subject_id, oktató_id, státusz `nem_elfogadott|elfogadott|átcsúszott`, forrás `token|admin`, időpont)
- `admin_decision` (entitás, típus, választott érték, admin, időpont) — a human-in-the-loop döntések
- `quarantine_row` (period_id, nyers sor, hibakód, ok)
- `email_token` (lásd D7), `audit_log` (ki, mi, mikor, forrás)

### D13 — Ingest folyamat
Laci feltölti a 2 Excelt → streamelt olvasás (ExcelJS) → szerződés-JOIN adóazonosító-hash-en → időszak-particionálás `Óra kezdete` szerint → validáció + karantén + admin-döntés jelölés → díj- és jogosultság-számítás → tételes commit (65k sor nem egy tranzakcióban). Nincs felülíró újrafeltöltés nyitott időszakon belül.

## Risks / Trade-offs

- **[Adóazonosító mint join-kulcs]** → titkosítás + HMAC-hash + teljes kizárás; hozzáférés audit-naplózva.
- **[Token = aláírás, most pénzt kapuz]** → erős entrópia + hash-tárolás + lezáráskori érvénytelenítés + metaadat-napló; a maradék jogi kockázat tudatosan vállalva (nem gyengébb a papírnál; a rendszer csak bérszámfejtési input).
- **[HR-adatminőség: párosítatlan/ütköző/e-mail nélküli sorok]** → admin-döntés + „kézbesíthetetlen" lista + HR-feedback; nem tippelünk.
- **[A/B nyitott a becsléskor]** → e-mail és forrás-DB interfész/fázis mögé; A/B beérkezésekor pontosítás.
- **[65k soros ingest teljesítmény]** → streamelt olvasás + tételes commit + indexelt JOIN-kulcs; Laci egyszeri művelete.

## Migration Plan

1. Séma létrehozása (migrations), titkosítási kulcs/KMS az adóazonosítóhoz.
2. Forrás-DB séma felmérése (B) → mező-megfeleltetés.
3. ETL staging-be **pillanatképként** (nincs újraszámolás), validáció (sorszám-egyezés, kulcs-feloldás), majd élesbe. Fázisolható a mag MVP után.
4. Rollback: az új rendszer írásvédett a forrásra; hiba esetén az új DB üríthető és az ETL újrajátszható (idempotens).

## Open Questions

- **A** — E-mail küldés mechanizmusa (IG/infra). Feltételezve: Graph sendMail, interfész mögött.
- **B** — Forrás-DB motor és séma (jelenlegi rendszergazda). Migráció fázisolható.
- **C** — Support-modell (IG + BA).
- **D** — GDPR-jogalap a nyers adóazonosító korlátlan megőrzéséhez (adatvédelmi/jogi).
- **E** — Tárgyfelelős (személy + e-mail) forrása: feltételezve, hogy az Excelből kiolvasható (ügyfélnek továbbítva).
- **F** — Nyelvi Intézet szaktantárgy-Tárgykód listája a besoroló táblához (ügyfél / Nyelvi Intézet).
