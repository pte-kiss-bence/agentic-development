## ADDED Requirements

### Requirement: A két HR-Excel feltöltése és JOIN-ja
A rendszer SHALL lehetővé tenni, hogy az admin együtt feltöltse a 2 HR-Excelt: az **óranyilvántartást** (foglalkozás-soronkénti oktatási adat) és a **szerződés-listát** (személyenkénti szerződés-státusz + e-mail). A rendszer SHALL a két fájlt a személy **adóazonosító**-kulcsán JOIN-olni (az óranyilvántartás `Adóazonosító` mezője = szerződés `adószám`), és a feloldható sorokból `assignment` rekordot képezni. A tárgy adata teljes egészében az óranyilvántartásból származik (nem külön fájlból).

#### Scenario: Sikeres feltöltés és JOIN
- **WHEN** az admin feltölti a 2 érvényes Excelt
- **THEN** a rendszer az óranyilvántartás sorait a szerződés-adattal (státusz, e-mail) az adóazonosító-hash kulcson JOIN-olja, és a feloldható sorokat `assignment` rekordként rögzíti

### Requirement: Időszak levezetése az óra dátumából, több-hónapos feltöltés
A rendszer SHALL minden sort az `Óra kezdete` mező **hónapja** szerint a megfelelő havi időszakhoz rendelni. Egy feltöltés SHALL több hónapot is tartalmazhat; ilyenkor a rendszer SHALL a sorokat hónap-időszakonként particionálni, és minden érintett hónaphoz a nyitott időszakot feltölteni (vagy létrehozni, ha még nem létezik).

#### Scenario: Egy feltöltés több hónapra
- **WHEN** a feltöltött óranyilvántartás két különböző hónapba eső sorokat tartalmaz
- **THEN** a rendszer a sorokat az `Óra kezdete` hónapja szerint a két külön havi időszakhoz rendeli

#### Scenario: Lezárt hónapba eső sor
- **WHEN** egy feltöltött sor `Óra kezdete`-je egy már lezárt (immutable) hónapba esik
- **THEN** a rendszer a lezárt időszakot NEM módosítja, a sort admin-döntésre jelöli, és a döntést (kiigazítás vagy elvetés) auditálja

### Requirement: Soronkénti validáció, karantén és eldönthetetlen sorok
A rendszer SHALL soronként validálni a feltöltött adatot (kötelező oszlopok megléte, forma, a JOIN-kulcs feloldhatósága), a hibás sorokat karanténba helyezni hibakóddal, a jó sorokat betölteni, és a betöltött, illetve karanténos sorok számát láthatóan megjeleníteni. Ahol egy korrektség-kritikus érték nem dönthető el determinisztikusan (pl. díjkategória, szerződés-státusz ütközés, párosítatlan oktató), a rendszer SHALL a sort az admin felületen **választható opcióval** megjeleníteni, és MUST NOT csendben alapértelmezni vagy mindent karanténba dobni.

#### Scenario: Hibás sor karanténba kerül
- **WHEN** egy sorból hiányzik kötelező mező, vagy a JOIN-kulcs nem oldható fel
- **THEN** a rendszer a sort `quarantine_row`-ként rögzíti hibakóddal, és NEM hoz létre belőle `assignment`-et

#### Scenario: Eldönthetetlen sor admin-döntésre
- **WHEN** egy sor betölthető, de egy korrektség-kritikus értéke (pl. díjkategória) nem dönthető el egyértelműen
- **THEN** a rendszer a sort betölti, de admin-döntésre jelöli egy felületi választható opcióval, és a levezetett díjat addig nem véglegesíti

#### Scenario: Karantén és betöltés látható számlálása
- **WHEN** a feldolgozás befejeződött
- **THEN** a rendszer megjeleníti a betöltött, a karanténos és az admin-döntésre váró sorok számát, és a listák az admin számára elérhetők

### Requirement: Egyszeri, végleges feltöltés időszakonként
A rendszer SHALL egy adott hónap-időszak adatát a feltöltésből véglegesnek tekinteni; ugyanazon nyitott időszakhoz újabb, teljes felülíró feltöltés MUST NOT legyen lehetséges. (A hiányzó/késői sorok kezelése az admin-döntési, illetve — lezárt időszaknál — a kiigazítási úton történik, nem újrafeltöltéssel.)

#### Scenario: Ismételt felülíró feltöltés tiltása
- **WHEN** egy nyitott hónap-időszak már fel lett töltve, és az admin ugyanazt az időszakot teljes egészében újra feltöltené
- **THEN** a rendszer elutasítja a felülíró feltöltést, és jelzi, hogy az időszak adata már rögzített

### Requirement: Adóazonosító védett kezelése feldolgozáskor
A rendszer SHALL az adóazonosítót **titkosítva tárolni** (megőrizve a nyers értéket a jövőbeli igényekre) és a JOIN/keresés céljára determinisztikus **HMAC-hash** kulcson illeszteni. A nyers adóazonosítót a rendszer SHALL kizárni minden nézetből, exportból, PDF-ből, tokenből és naplóból; hozzáférés csak az ingest/migrációs komponensnek, audit-naplózva.

#### Scenario: Adóazonosító nem szivárog
- **WHEN** a feldolgozás naplóz, vagy az admin megnyitja a karantén/eredmény nézetet
- **THEN** a nyers adóazonosító egyik felületen, exportban, PDF-ben vagy naplóbejegyzésben sem jelenik meg

#### Scenario: Illesztés a nyers érték kitétele nélkül
- **WHEN** a rendszer a személyt a szerződés-adattal illeszti
- **THEN** az illesztés a determinisztikus HMAC-hash kulcson történik, a nyers adóazonosító felfedése nélkül
