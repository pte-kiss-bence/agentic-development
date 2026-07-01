# Az időszakot az óra dátuma dönti el, és több nyitott időszak lehet

Egy foglalkozás időszakhoz (naptári hónaphoz) tartozását **mindig az `Óra kezdete` hónapja** dönti el, és **egyszerre több nyitott időszak** is létezhet. Egy HR-feltöltés több hónap adatát is tartalmazhatja, ezért a rendszer a sorokat hónaponként particionálja; az admin az időszakokat egyenként, manuálisan zárja.

## Considered Options

- *„Egyszerre egy nyitott időszak / egy feltöltés per időszak"* (az eredeti openspec-terv) — **elvetve**, mert a valós feltöltés több hónapba eső sorokat is hoz, így nem tartható.

## Consequences

- Lezárt (immutable) hónapba eső, később érkező sor nem módosítja a lezárt időszakot, hanem **admin-döntésre** kerül (korrekció vagy elvetés), auditáltan.
