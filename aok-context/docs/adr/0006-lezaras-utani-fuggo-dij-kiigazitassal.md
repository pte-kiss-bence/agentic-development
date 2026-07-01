# Lezárás utáni függő díj kiigazításként, a következő időszakban

Ha egy lezárt (immutable) időszakban átcsúszottként maradt **függő díj** később felszabadul, az **nem** a lezárt időszakban, hanem egy **kiigazítás** tételként a **következő nyitott időszakban** fizetődik ki, az eredeti (időszak, tárgy, oktató) tételre hivatkozva. A lezárt időszak PDF-je és aggregátuma befagyva marad. Mivel lezáráskor minden token érvénytelenné válik, a rendezés lezárás után **kizárólag admin override-dal** történik (a tárgyfelelős offline jelzését az admin rögzíti) — nem adunk ki új tokent.

## Considered Options

- *A lezárt időszak felülírása* az utólagos elfogadáskor — **elvetve**, sérti az immutabilitást (ADR 0002).
- *Lezárás előtt kötelező minden függőt rendezni* — **elvetve**, a tárgyfelelős késése blokkolná a havi lezárást (a HR-késés amúgy is fő ütemkockázat).

## Consequences

- Az oktató a felszabadított díjat a következő havi elszámolásban kapja, „<eredeti hónap> kiigazítás" megjelöléssel. Kell egy kiigazítás-tétel, amely egy lezárt időszak elemére hivatkozik, és az auditban követhető.
