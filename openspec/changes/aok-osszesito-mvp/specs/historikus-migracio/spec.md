## ADDED Requirements

### Requirement: Historikus adat importja változtathatatlan pillanatképként
A rendszer SHALL a jelenlegi rendszer adatbázisából egy egyszeri, idempotens migrációval betölteni a historikus időszakok adatait az új sémába, **read-only pillanatképként**. A migráció SHALL a régi rendszer által tárolt eredményt (órák, státuszok, és díj, ha van) átvenni, és MUST NOT a mai díjmotorral újraszámolni a historikus díjakat. A migráció MUST NOT írjon a forrásrendszerbe, és csak olvasásra férjen hozzá.

#### Scenario: Historikus díj nem számolódik újra
- **WHEN** a migráció egy régi, lezárt időszakot importál, amelyhez a forrásban díjösszeg tartozik
- **THEN** a rendszer a forrásbeli díjat változatlanul, pillanatképként veszi át, és nem alkalmazza rá a mai díjmotort

#### Scenario: Régi rendszer nem tárolt díjat
- **WHEN** a forrásrendszer egy időszakhoz nem tárol díjat
- **THEN** a historikus nézet a meglévő adatot (óra/elfogadás) mutatja, kitalált díj nélkül

#### Scenario: Idempotens újrajátszás
- **WHEN** a migrációt kétszer futtatják ugyanazon forrásadattal
- **THEN** az eredmény azonos, és nem keletkeznek duplikált rekordok

### Requirement: Migrált adat megfeleltetése az új kulcsokhoz
A rendszer SHALL a migrált személyeket és tárgyakat ugyanazon kulcsokon azonosítani, mint az Excel-ingest (személy: adóazonosító-hash), hogy a historikus és az új adat konzisztensen összeálljon. A nem feloldható forrás-rekordokat SHALL migrációs hibalistában megjelölni. Mivel a migráció csak megjelenítést szolgál és a forrás-séma ismerete nyitott (Open Question), a capability a mag MVP után **fázisolható**.

#### Scenario: Feloldhatatlan forrás-rekord jelölése
- **WHEN** egy forrás-rekord kulcsa nem feleltethető meg az új séma kulcsainak
- **THEN** a rendszer a rekordot migrációs hibalistába teszi, és nem hoz létre belőle hibás összerendelést
