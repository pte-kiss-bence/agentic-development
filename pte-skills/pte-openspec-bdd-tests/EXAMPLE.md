# Worked example: OpenSpec spec → Gherkin

One end-to-end transformation showing the non-obvious moves: adding the missing `Given`, lifting a shared precondition into `Background`, collapsing data-only variants into a `Scenario Outline`, and preserving the spec's language with `# language: hu`.

## Input — OpenSpec spec

`openspec/specs/cikk-hozzaferes/spec.md`

```markdown
### Requirement: Előfizetési szint szerinti cikkhozzáférés
A rendszer SHALL az előfizetés szintje szerint engedélyezze a cikkekhez való hozzáférést.

#### Scenario: Ingyenes előfizető csak ingyenes cikket lát
- **WHEN** ingyenes előfizetésű felhasználó bejelentkezik
- **THEN** a rendszer megjeleníti az ingyenes cikket
- **THEN** a rendszer elrejti a fizetős cikket

#### Scenario: Fizető előfizető a fizetős cikket is látja
- **WHEN** fizetős előfizetésű felhasználó bejelentkezik
- **THEN** a rendszer megjeleníti a fizetős cikket is
```

## Output — Gherkin feature file

`features/cikk-hozzaferes/elofizetesi-szint-szerinti-cikkhozzaferes.feature`

```gherkin
# language: hu
# Spec: cikk-hozzaferes › Előfizetési szint szerinti cikkhozzáférés
Jellemző: Előfizetési szint szerinti cikkhozzáférés

  A rendszer az előfizetés szintje szerint enged hozzáférést a cikkekhez.

  Szabály: A hozzáférhető cikkek körét az előfizetés szintje dönti el

    Háttér:
      Adott egy regisztrált felhasználó

    Forgatókönyv vázlat: Az előfizető a szintjének megfelelő cikkeket látja
      Adott "<szint>" szintű előfizetése van
      Amikor bejelentkezik az érvényes hitelesítő adataival
      Akkor látja az ingyenes cikket
      És a fizetős cikket <fizetos>

      Példák:
        | szint    | fizetos   |
        | ingyenes | nem látja |
        | fizetős  | látja     |
```

## Why each move

- **`# language: hu`** — the spec is Hungarian, so the Gherkin keywords (`Jellemző`, `Szabály`, `Háttér`, `Forgatókönyv vázlat`, `Adott`/`Amikor`/`Akkor`/`És`, `Példák`) come from the gherkin i18n set for that language.
- **Traceability comment** — `# Spec: <capability> › <Requirement>` links the feature back to its source.
- **`Szabály` (Rule)** carries the OpenSpec Requirement; the `Jellemző` (Feature) carries the capability.
- **`Háttér` (Background)** — neither scenario stated a precondition, but both assume a registered user. The `Given` is supplied once here instead of repeated.
- **`Forgatókönyv vázlat` (Scenario Outline)** — the two OpenSpec scenarios differ only in subscription level and what is visible, so they collapse into one outline with an `Példák` (Examples) table. Two genuinely different behaviours would have stayed two scenarios.
- **Declarative wording** — "bejelentkezik az érvényes hitelesítő adataival", not "kitölti az e-mail mezőt és megnyomja a Belépés gombot". The behaviour survives a UI redesign.
