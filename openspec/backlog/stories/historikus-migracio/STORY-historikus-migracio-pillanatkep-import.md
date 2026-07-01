# Historikus adat read-only pillanatkép-importja · STORY-historikus-migracio-pillanatkep-import

Parent epic: `EPIC-historikus-migracio`

## Description
Mint **HR/vezető**, szeretném **a historikus időszakokat read-only pillanatképként az új rendszerben látni**, hogy **a régi eredmények elérhetők maradjanak újraszámolás nélkül**.

Egy egyszeri, idempotens, csak-olvasó DB→DB migráció a régi eredményt (órák, státuszok, díj, ha van) változatlan pillanatképként veszi át; a mai díjmotorral nem számol újra, és nem ír a forrásrendszerbe. Ahol a forrás nem tárolt díjat, a nézet a meglévő adatot mutatja, kitalált díj nélkül. Kétszeri futtatás azonos eredmény, duplikátum nélkül.

## Context
Friss start megtartott múlttal: a historikus adat megjelenítése, nem újraértelmezése.

## BDD Test
```gherkin
# language: hu
# Spec: historikus-migracio › Historikus adat importja változtathatatlan pillanatképként (change: aok-osszesito-mvp)
@EPIC-historikus-migracio @STORY-historikus-migracio-pillanatkep-import
Jellemző: historikus-migracio
  Egyszeri, idempotens, read-only migráció; a historikus díjat nem számolja újra.

  Szabály: Historikus adat importja változtathatatlan pillanatképként

    Forgatókönyv: Historikus díj nem számolódik újra
      Adott egy régi, lezárt időszak a forrásban tárolt díjösszeggel
      Amikor a migráció importálja
      Akkor a rendszer a forrásbeli díjat változatlanul, pillanatképként veszi át
      És nem alkalmazza rá a mai díjmotort

    Forgatókönyv: Régi rendszer nem tárolt díjat
      Adott egy forrás-időszak, amelyhez nem tartozik díj
      Amikor a migráció importálja
      Akkor a historikus nézet a meglévő adatot (óra/elfogadás) mutatja
      És nem talál ki díjat

    Forgatókönyv: Idempotens újrajátszás
      Adott ugyanaz a forrásadat
      Amikor a migrációt kétszer futtatják
      Akkor az eredmény azonos
      És nem keletkeznek duplikált rekordok
```

## Estimation
- **3-Points becslés (ideális óra):** O 8 / M 14 / P 44 | **Eβ 18**, σ 6
- **Becsült munkaóra:** 18 ó (≈ 3 ideális nap)
- **Story Point:** 8

## Risks and Dependencies
- Függőség: a jelenlegi rendszer forrás-DB sémája (Open Question B) — enélkül a migráció feltételes; read-only hozzáférés a forráshoz.
- Kockázat: a forrás-séma ismeretének hiánya miatt a story fázisolható a mag MVP után (magas becslési bizonytalanság).
