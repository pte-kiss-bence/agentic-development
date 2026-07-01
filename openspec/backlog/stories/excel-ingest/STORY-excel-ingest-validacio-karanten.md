# Validáció, karantén és admin-döntésre jelölés · STORY-excel-ingest-validacio-karanten

Parent epic: `EPIC-excel-ingest`

## Description
Mint **admin**, szeretném **a hibás sorokat karanténban, az eldönthetetleneket admin-döntésre látni számlálóval**, hogy **sosem tippel a rendszer csendben, és lássam a feldolgozás eredményét**.

A rendszer soronként validál (kötelező oszlopok, forma, JOIN-kulcs feloldhatósága), a hibás sort karanténba teszi hibakóddal, a korrektség-kritikus, nem determinisztikus értéket (pl. díjkategória) admin-döntésre jelöli választható opcióval, és láthatóan megjeleníti a betöltött / karantén / admin-döntésre váró sorszámokat.

## Context
Adatminőségi kapu: a rossz sor nem szennyezi a díjszámítást, az eldönthetetlen pedig emberi döntésre vár.

## BDD Test
```gherkin
# language: hu
# Spec: excel-ingest › Soronkénti validáció, karantén és eldönthetetlen sorok (change: aok-osszesito-mvp)
@EPIC-excel-ingest @STORY-excel-ingest-validacio-karanten
Jellemző: excel-ingest
  Soronkénti validáció: hibás sor karanténba, eldönthetetlen admin-döntésre, látható számlálással.

  Szabály: Soronkénti validáció, karantén és eldönthetetlen sorok

    Forgatókönyv: Hibás sor karanténba kerül
      Adott egy feltöltött sor, amelyből hiányzik kötelező mező vagy a JOIN-kulcs nem oldható fel
      Amikor a rendszer validálja a sort
      Akkor a sort quarantine_row-ként rögzíti hibakóddal
      És nem hoz létre belőle assignment-et

    Forgatókönyv: Eldönthetetlen sor admin-döntésre
      Adott egy betölthető sor, amelynek egy korrektség-kritikus értéke nem dönthető el egyértelműen
      Amikor a rendszer feldolgozza a sort
      Akkor betölti, de admin-döntésre jelöli választható opcióval
      És a levezetett díjat addig nem véglegesíti

    Forgatókönyv: Karantén és betöltés látható számlálása
      Adott egy befejezett feldolgozás
      Amikor az admin megnyitja az eredmény nézetet
      Akkor a rendszer megjeleníti a betöltött, a karanténos és az admin-döntésre váró sorok számát
      És a listák elérhetők az admin számára
```

## Estimation
- **3-Points becslés (ideális óra):** O 12 / M 18 / P 34 | **Eβ 20**, σ 3,7
- **Becsült munkaóra:** 20 ó (≈ 3,3 ideális nap)
- **Story Point:** 8

## Risks and Dependencies
- Függőség: `STORY-excel-ingest-feltoltes-join` (a validálandó sorok), `STORY-belso-hozzaferes-admin-feloldas` (az admin-döntés feloldó felülete).
- Kockázat: HR-adatminőség (~11,8% párosítatlan, ~23% e-mail nélkül) → nagy admin-döntési és karantén-volumen.
