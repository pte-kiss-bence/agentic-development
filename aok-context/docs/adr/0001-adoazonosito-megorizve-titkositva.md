# Az adóazonosítót megőrizve, titkosítva tároljuk (nem hash-only)

Az adóazonosító (a két HR-Excel JOIN-kulcsa, érzékeny nemzeti azonosító) nyers értékét **megőrizzük, titkosítva** tároljuk, és a JOIN/keresés determinisztikus HMAC-hash kulcson megy. Bár a GDPR-adatminimalizálás a „csak hash-t tárolunk, nyerset nem" megoldást sugallná, az ügyfél jövőbeli igénye miatt (az adóazonosító visszakereshető legyen; ma is az Excelben van) a nyers értéket megtartjuk — minden nézetből, PDF-ből, exportból, tokenből és naplóból kizárva, hozzáférés csak az ingest/migráció komponensnek, audit-naplózva.

## Consequences

- Élesíti a nyers adóazonosító **korlátlan megőrzésének GDPR-jogalapját** (Open Question D). Ha a jogi/adatvédelmi vélemény felső korlátot ír elő, utólag törlő/anonimizáló mechanizmust kell építeni.
