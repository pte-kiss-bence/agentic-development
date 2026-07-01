# Saját-elfogadás engedélyezése és jelölése · STORY-targyfelelos-elfogadas-sajat-elfogadas

Parent epic: `EPIC-targyfelelos-elfogadas`

## Description
Mint **auditor**, szeretném **a saját-elfogadást engedni, de külön jelölni és szűrhetővé tenni**, hogy **az összeférhetetlenség átlátható maradjon**.

A tárgyfelelős a saját, oktatóként megtartott óráit is elfogadhatja (saját-elfogadás); a rendszer ezt nem tiltja, de az auditban self-approval jelzővel megjelöli, és a belső táblázat-nézetben szűrhetővé teszi — jelöletlenül nem hagyja.

## Context
Engedett, de átláthatóan jelölt összeférhetetlenség: a tárgyfelelős a saját kifizetését kapuzza.

## BDD Test
```gherkin
# language: hu
# Spec: targyfelelos-elfogadas › Saját-elfogadás engedélyezése és jelölése (change: aok-osszesito-mvp)
@EPIC-targyfelelos-elfogadas @STORY-targyfelelos-elfogadas-sajat-elfogadas
Jellemző: targyfelelos-elfogadas
  A saját-elfogadás engedett, de auditban self-approval jelzővel jelölt és szűrhető.

  Szabály: Saját-elfogadás engedélyezése és jelölése

    Forgatókönyv: Tárgyfelelős elfogadja a saját óráit
      Adott egy (tárgy × oktató) sor, ahol az oktató maga a tárgyfelelős
      Amikor a tárgyfelelős elfogadja ezt a sort
      Akkor a rendszer az elfogadást rögzíti
      És saját-elfogadás jelzővel naplózza
      És a belső nézetben szűrhetővé teszi
```

## Estimation
- **3-Points becslés (ideális óra):** O 3 / M 5 / P 9 | **Eβ 5**, σ 1
- **Becsült munkaóra:** 5 ó (≈ 0,8 ideális nap)
- **Story Point:** 3

## Risks and Dependencies
- Függőség: `STORY-belso-hozzaferes-tablazat-nezet` (a szűrhetőség), `STORY-belso-hozzaferes-audit-naplo` (a jelző naplózása).
- Kockázat: a self-approval jelző elmaradása → rejtett önjóváhagyás.
