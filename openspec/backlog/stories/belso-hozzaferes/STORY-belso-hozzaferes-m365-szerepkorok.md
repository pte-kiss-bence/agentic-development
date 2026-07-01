# M365 belépés és szerepkörök · STORY-belso-hozzaferes-m365-szerepkorok

Parent epic: `EPIC-belso-hozzaferes`

## Description
Mint **belső felhasználó**, szeretném **M365-tel belépni és a szerepkörömnek megfelelő jogot kapni**, hogy **csak az admin szerkeszthessen, a reader pedig biztonságosan csak olvasson**.

Entra ID (M365 / OIDC) hitelesítés MSAL + JWT-validációval, két szerepkörrel: `admin` (feltöltés, admin-döntés, override, lezárás) és `reader` (csak olvas). A `reader` szerkesztő művelete szerver oldalon megtagadva.

## Context
A belső hozzáférés alapja: szerver oldalon kikényszerített szerepkör-szeparáció.

## BDD Test
```gherkin
# language: hu
# Spec: belso-hozzaferes › M365 alapú belső hozzáférés és szerepkörök (change: aok-osszesito-mvp)
@EPIC-belso-hozzaferes @STORY-belso-hozzaferes-m365-szerepkorok
Jellemző: belso-hozzaferes
  M365 hitelesítés és szerepkör-szeparáció (admin szerkeszt, reader olvas).

  Szabály: M365 alapú belső hozzáférés és szerepkörök

    Forgatókönyv: Admin szerkeszthet
      Adott egy admin szerepkörű, M365-tel hitelesített felhasználó
      Amikor belép a rendszerbe
      Akkor elérhetők számára a feltöltés, az admin-döntés/override és a lezárás műveletek

    Forgatókönyv: Reader csak olvas
      Adott egy reader szerepkörű, M365-tel hitelesített felhasználó
      Amikor egy szerkesztő műveletet próbál indítani
      Akkor a rendszer megtagadja a műveletet
      És nem módosít adatot
```

## Estimation
- **3-Points becslés (ideális óra):** O 4 / M 8 / P 16 | **Eβ 9**, σ 2
- **Becsült munkaóra:** 9 ó (≈ 1,5 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: Entra ID (M365/OIDC) tenant és szerepkör-hozzárendelés (ADR 0005).
- Kapcsolódás: minden szerkesztő művelet (`EPIC-excel-ingest`, override, lezárás) erre a szerepkör-ellenőrzésre épül.
- Kockázat: téves szerepkör-hozzárendelés → jogosulatlan szerkesztés.
