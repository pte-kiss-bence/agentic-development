# Tasks — ÁOK Oktató- és tárgyfelelős összesítő (MVP)

> Prioritási sorrend a korrektség-mag köré bázisolva. Jelölés: ⚠️A = e-mail mechanizmus (Open A) · ⚠️B = forrás-DB (Open B) · ⚠️E = tárgyfelelős-forrás (Open E) · ⚠️F = Nyelvi Intézet szaktantárgy-lista (Open F). A feltételes pontok addig becslés-szinten szerepelnek. Stack: NestJS + Angular + PostgreSQL.

## 1. Alapozás és infrastruktúra

- [ ] 1.1 Solution/repo váz (NestJS backend + Angular frontend), CI alap, Docker/konténer
- [ ] 1.2 PostgreSQL + migrations alap
- [ ] 1.3 Entra ID / OIDC bekötése (MSAL frontend + JWT-validáció backend), `admin` és `reader` szerepkör
- [ ] 1.4 Titkosítási kulcs / KMS az adóazonosítóhoz (app-szintű AES) + HMAC-hash kulcs
- [ ] 1.5 Naplózás (audit_log) infrastruktúra és középréteg

## 2. Adatmodell

- [ ] 2.1 `person` (titkosított adóazonosító + `adoazonosito_hash`, e-mail, szerződés-státusz)
- [ ] 2.2 `subject` (tárgykód, megnevezés, szervezeti egység, díjkategória-felülírás) ⚠️F
- [ ] 2.3 `period` (havi, státusz, referencia-azonosító), **több nyitott** támogatása
- [ ] 2.4 `assignment` (kurzustípus, nyelv, tanóra, számított díj, jogosultság, díj-státusz), `acceptance_status`
- [ ] 2.5 `admin_decision` (human-in-the-loop döntések), `quarantine_row`, `email_token`

## 3. Excel-ingest (capability: excel-ingest)

- [ ] 3.1 Excel-feltöltő felület (admin), 2 fájl együtt
- [ ] 3.2 Streamelt olvasás (ExcelJS), 65k sor, tételes commit
- [ ] 3.3 JOIN a szerződés-fájllal az adóazonosító-hash kulcson (státusz + e-mail feloldás)
- [ ] 3.4 Időszak-particionálás az `Óra kezdete` hónapja szerint (több-hónapos feltöltés)
- [ ] 3.5 Soronkénti validáció (kötelező oszlopok, forma, kulcs-feloldás), karantén + számláló
- [ ] 3.6 Eldönthetetlen sorok admin-döntésre jelölése (nem csendes default)
- [ ] 3.7 Lezárt hónapba eső sor → admin-döntés (a lezárt időszak érintése nélkül)
- [ ] 3.8 Felülíró újrafeltöltés tiltása nyitott időszakon belül
- [ ] 3.9 Adóazonosító-kizárás minden nézetből/exportból/PDF/logból (teszt is)

## 4. Díjszámítás (capability: dij-szamitas)

- [ ] 4.1 Tanóra-számítás (`Órahossz/45`) és foglalkozás-alapú díj (tétel × tanóra), csak `Nemindul`=Hamis
- [ ] 4.2 Díjkategória-levezetés: `Kurzustípus` default + tárgy-szintű besoroló referenciatábla (testnevelés/nyelvi) ⚠️F
- [ ] 4.3 Nyelvi Intézet: Tárgykód-kivétellista (szaktantárgy vs nyelvi), eldönthetetlen → admin ⚠️F
- [ ] 4.4 Jogosultság: szerződés-`státusz` × `Kurzus nyelve` szabály
- [ ] 4.5 Szerződés-státusz feloldás + tie-break (több szerződés, ütközés jelölése)
- [ ] 4.6 Párosítatlan oktató → admin-döntés + függő díj + HR-hiánylista
- [ ] 4.7 Fizetendő vs függő díj szétválasztása (kifizetés az elfogadáshoz kötve)
- [ ] 4.8 Díjmotor bővíthető absztrakció (kategória → elszámolási alap → tétel), MVP csak foglalkozás-alap

## 5. Tárgyfelelős-elfogadás (capability: targyfelelos-elfogadas)

- [ ] 5.1 Token generálás (≥128 bit), hash-tárolás, `(időszak, tárgyfelelős)` kötés
- [ ] 5.2 Token-nézet: login nélküli, scope-olt, **(tárgy × oktató) soronkénti óra**-lista (díj nélkül)
- [ ] 5.3 Soronkénti „Elfogadom" — végleges, egyirányú; státusz + forrás írása; kifizetési kapu
- [ ] 5.4 Lejárt/lezárt időszak tokenjének elutasítása
- [ ] 5.5 Admin override az elfogadási státuszra (naplózva, díjat felszabadít)
- [ ] 5.6 Elfogadás-metaadat naplózása; audit jelzi, hogy token ≠ erős azonosítás
- [ ] 5.7 Adóazonosító és forint kizárása a token-nézetből/URL-ből (teszt is)

## 6. Értesítés

- [ ] 6.1 „Értesítések kiküldése" művelet (admin indítja, karantén/admin-döntés után)
- [ ] 6.2 ⚠️A E-mail küldés interfész mögött (Graph sendMail feltételezve), token-link a levélben ⚠️E
- [ ] 6.3 Oktatói statikus melléklet (saját óra + saját díj) az e-mailhez
- [ ] 6.4 ⚠️A Tömeges küldés: throttling, retry, kézbesítési státusz naplózása
- [ ] 6.5 Hiányzó e-mail → „kézbesíthetetlen" lista; tárgyfelelősnél admin override út

## 7. Lezárás és PDF (capability: idoszak-lezaras)

- [ ] 7.1 Manuális lezárás művelet (admin), több nyitott időszakból egyenként
- [ ] 7.2 Átcsúszott-jelölés + függő díj a `nem_elfogadott` sorokra lezáráskor
- [ ] 7.3 Blokkoló figyelmeztetés rendezetlen karanténra / admin-döntésre lezárás előtt
- [ ] 7.4 Tokenek érvénytelenítése lezáráskor; lezárt időszak immutabilitása
- [ ] 7.5 Áttekintő PDF: aggregált óra/díj + átcsúszott/függő kiemelés + visszakereső azonosító, adóazonosító nélkül

## 8. Belső nézet és jogosultság (capability: belso-hozzaferes)

- [ ] 8.1 Táblázat-nézet: oktató × tárgy × óra × díj
- [ ] 8.2 Szűrés (átcsúszott, függő, státusz, nyelv, tárgyfelelős) és aggregáció
- [ ] 8.3 Admin-feloldó felület az eldönthetetlen sorokra (választható opció, auditált döntés)
- [ ] 8.4 Szerepkör-érvényesítés: `reader` nem szerkeszthet
- [ ] 8.5 Audit napló nézet/lekérdezés (visszakövethetőség)

## 9. Migráció (capability: historikus-migracio) — fázisolható

- [ ] 9.1 ⚠️B Forrás-DB séma felmérése és mező-megfeleltetés
- [ ] 9.2 ⚠️B Idempotens ETL (csak olvasás), **pillanatkép-import, nincs újraszámolás**, staging → éles
- [ ] 9.3 Kulcs-megfeleltetés (adóazonosító-hash), feloldhatatlanok hibalistája
- [ ] 9.4 Migráció validáció (sorszám-egyezés, mintavétel)

## 10. Nem funkcionális és átadás

- [ ] 10.1 Teljesítmény-teszt: 65k ingest + ~100 egyidejű token-nézet
- [ ] 10.2 GDPR-ellenőrzés: adóazonosító titkosítás + kizárás end-to-end audit
- [ ] 10.3 Korrektség-tesztek: díj-, kategória-, jogosultság-levezetés valós mintán (a mag)
- [ ] 10.4 Naplózás teljességének ellenőrzése (minden lényegi művelet)
- [ ] 10.5 Tesztelési kör (07.27 feature freeze után), bugfix
- [ ] 10.6 Átadás-kész állapot (08.15), telepítési + rollback leírás
