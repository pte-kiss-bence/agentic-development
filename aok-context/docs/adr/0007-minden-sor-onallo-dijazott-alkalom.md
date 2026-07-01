# Minden sor önálló, díjazott oktatási alkalom (társtanításnál nincs osztás)

Az óranyilvántartás **minden sora egy önálló, díjazott oktatási alkalom**, és a fizetett oktató a soronkénti `Oktató neve`/`Adóazonosító` (nem a kurzus teljes névsora, a `Kurzushoz rendelt oktatók`, ami csak információ). Ha ugyanazt a `(Kurzus azonosító + Óra kezdete)` slotot több oktató tartja, az **oktatónként külön, teljes hosszú sorként** jelenik meg, és **mindegyik a teljes díjat kapja** — a slot díját NEM osztjuk el.

## Considered Options

- *Slot-szintű osztás* (egy foglalkozásért összesen egyszer jár a díj, elosztva a társ-oktatók között) — **elvetve**. A forrás oktatónként külön, teljes sort emel ki, és a felhasználó megerősítette, hogy „minden sor egy oktatási alkalmat jelöl"; nincs is megbízható osztási kulcs.

## Consequences

- Egy társtanított slot a résztvevők számával **többszörös** díjat termel (pl. egy 45 perces előadás két oktatóval 2 × 21 000 = 42 000 Ft). A mintában 6 263 ilyen többoktatós slot van.
- Egy jövőbeli fejlesztő ezt könnyen „duplikációnak" nézheti — **ne** vezessen be slot-deduplikációt; ez szándékos.
