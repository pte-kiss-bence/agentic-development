# Worked example: story card → embedded Gherkin

One end-to-end transformation showing the non-obvious moves: reading a story card's *Forrás-hivatkozás* row, tracing its `#### Scenario`s back to the spec, adding the missing `Given`, collapsing data-only variants into a `Scenario Outline`, tagging the block with the card's own trace IDs, and writing it into the card's `BDD teszt` section — in place, touching nothing else.

## Input — the story card (from `pte-openspec-to-stories`)

`stories/elofizetes-cikkhozzaferes/elofizetesi-szint-szerinti-cikkhozzaferes.md`, with its `BDD teszt` section still a placeholder:

```markdown
## Elfogadási kritériumok
- Amennyiben a felhasználónak ingyenes előfizetése van,
  Amikor bejelentkezik,
  Akkor a rendszer megjeleníti az ingyenes cikket,
  És elrejti a fizetős cikket.
- Amennyiben a felhasználónak fizetős előfizetése van,
  Amikor bejelentkezik,
  Akkor a rendszer megjeleníti a fizetős cikket is.

## BDD teszt
_(kitölti a `pte-openspec-bdd-tests`)_
```

Its *Forrás-hivatkozás* row names the trace IDs and the covered Scenarios:

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| `STORY-elofizetes-cikkhozzaferes-elofizetesi-szint-szerinti-cikkhozzaferes` | `EPIC-elofizetes-cikkhozzaferes` | Előfizetési szint szerinti cikkhozzáférés | Ingyenes előfizető csak ingyenes cikket lát; Fizető előfizető a fizetős cikket is látja |

The two `#### Scenario`s named in that row, traced to `openspec/specs/cikk-hozzaferes/spec.md`:

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

## Output — the same card, `BDD teszt` section filled in place

Only the `BDD teszt` section changes; every other section of the card is left untouched.

````markdown
## BDD teszt
```gherkin
# language: hu
# Spec: cikk-hozzaferes › Előfizetési szint szerinti cikkhozzáférés
@EPIC-elofizetes-cikkhozzaferes @STORY-elofizetes-cikkhozzaferes-elofizetesi-szint-szerinti-cikkhozzaferes
Jellemző: Előfizetési szint szerinti cikkhozzáférés

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
````

## Why each move

- **The card is the input, not the spec** — the story set, IDs, and Scenario split already exist. This skill reads the card's *Forrás-hivatkozás* row for **which** Scenarios and **which** trace IDs, then traces to the spec only for the `WHEN`/`THEN` behaviour. It mints nothing (see `CONVENTIONS.md`).
- **`@EPIC-…`/`@STORY-…` tags** — lifted verbatim from the card's row, so the epic, the card, and the embedded Gherkin all key off the same IDs.
- **`# language: hu`** — the spec is Hungarian, so the Gherkin keywords (`Jellemző`, `Szabály`, `Háttér`, `Forgatókönyv vázlat`, `Adott`/`Amikor`/`Akkor`/`És`, `Példák`) come from the gherkin i18n set for that language.
- **`Szabály` (Rule)** carries the card's source Requirement; the `Jellemző` (Feature) carries the capability.
- **`Háttér` (Background)** — neither scenario stated a precondition, but both assume a registered user. The `Given` is supplied once here instead of repeated.
- **`Forgatókönyv vázlat` (Scenario Outline)** — the card's two Scenarios differ only in subscription level and what is visible, so they collapse into one outline with a `Példák` (Examples) table. Two genuinely different behaviours would have stayed two scenarios.
- **Declarative wording** — "bejelentkezik az érvényes hitelesítő adataival", not "kitölti az e-mail mezőt és megnyomja a Belépés gombot". The behaviour survives a UI redesign.
- **In place, nothing else touched** — only the `BDD teszt` placeholder is replaced; the acceptance criteria, INVEST check, and trace row stay byte-for-byte.
