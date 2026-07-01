# Worked example: Agile Epic story → refinement-ready story card

One end-to-end transformation showing the non-obvious moves: taking a single story out of an epic's *User story-k* list, reusing its `STORY-…`/`EPIC-…` IDs and `Mint …` line verbatim, tracing the map's `#### Scenario`s back to the spec, and expanding their `WHEN`/`THEN` into `Amennyiben`/`Amikor`/`Akkor` acceptance criteria with a supplied precondition.

## Input — one story from the epic

From `epics/elofizetes-cikkhozzaferes.md` (produced by `pte-openspec-to-epics`):

```markdown
## User story-k
- `STORY-elofizetes-cikkhozzaferes-elofizetesi-szint-szerinti-cikkhozzaferes`
  Mint olvasó, szeretném a szintemnek megfelelő cikkeket látni, hogy csak a
  jogosult tartalomhoz férjek hozzá.
  - Ingyenes előfizető az ingyenes cikket látja, a fizetőset nem.
  - Fizető előfizető a fizetős cikket is látja.
```

Its *Forrás-spec hivatkozás* row:

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| `STORY-elofizetes-cikkhozzaferes-elofizetesi-szint-szerinti-cikkhozzaferes` | `EPIC-elofizetes-cikkhozzaferes` | Előfizetési szint szerinti cikkhozzáférés | Ingyenes előfizető csak ingyenes cikket lát; Fizető előfizető a fizetős cikket is látja |

The two `#### Scenario`s named in that row, traced to `openspec/specs/cikk-hozzaferes/spec.md`:

```markdown
#### Scenario: Ingyenes előfizető csak ingyenes cikket lát
- **WHEN** ingyenes előfizetésű felhasználó bejelentkezik
- **THEN** a rendszer megjeleníti az ingyenes cikket
- **THEN** a rendszer elrejti a fizetős cikket

#### Scenario: Fizető előfizető a fizetős cikket is látja
- **WHEN** fizetős előfizetésű felhasználó bejelentkezik
- **THEN** a rendszer megjeleníti a fizetős cikket is
```

## Output — story card file

`stories/elofizetes-cikkhozzaferes/elofizetesi-szint-szerinti-cikkhozzaferes.md`

```markdown
# Cím
Szintnek megfelelő cikkhozzáférés —
`STORY-elofizetes-cikkhozzaferes-elofizetesi-szint-szerinti-cikkhozzaferes`
(`EPIC-elofizetes-cikkhozzaferes`)

## User story
Mint olvasó, szeretném a szintemnek megfelelő cikkeket látni, hogy csak a
jogosult tartalomhoz férjek hozzá.

## Kontextus
Az olvasó előfizetési szintje dönti el, mely cikkeket látja. Ez a történet a
hozzáférés megjelenítését szeleteli ki — az előfizetés indítása és lemondása
külön történet.

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

## INVEST-ellenőrzés
Independent ✓ · Negotiable ✓ · Valuable ✓ · Estimable ✓ · Small ✓ · Testable ✓
— önállóan szállítható; a mérhető kimenet a megjelenített cikkek köre.

## Készenléti feltétel (DoR)
_(csapat tölti ki refinementen)_

## Elkészültségi feltétel (DoD)
_(csapat tölti ki refinementen)_

## Prioritás
_(csapat tölti ki)_

## Becslés
_(story point — refinementen becsülve)_

## Függőségek és kockázatok
- Függ a hitelesítéstől (bejelentkezett felhasználó, ismert előfizetési szint).
- Testvér-történetek:
  `STORY-elofizetes-cikkhozzaferes-elofizetes-kezelese-inditas`,
  `STORY-elofizetes-cikkhozzaferes-elofizetes-kezelese-lemondas`.

## Forrás-hivatkozás
| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| `STORY-elofizetes-cikkhozzaferes-elofizetesi-szint-szerinti-cikkhozzaferes` | `EPIC-elofizetes-cikkhozzaferes` | Előfizetési szint szerinti cikkhozzáférés | Ingyenes előfizető csak ingyenes cikket lát; Fizető előfizető a fizetős cikket is látja |
```

## Why each move

- **Epic is the input, not the spec** — the story set and IDs already exist; this skill only expands. The `STORY-…`/`EPIC-…` IDs and the `Mint …` line are reused **verbatim** — this skill mints nothing (see `CONVENTIONS.md`).
- **Acceptance criteria traced 1:1** — each Scenario's `WHEN` becomes `Amikor`, its `THEN` bullets become `Akkor` + `És`. The first Scenario's two `THEN`s (megjeleníti / elrejti) map to `Akkor` + `És` — no outcome dropped, none added.
- **Supplied `Amennyiben`** — the spec states the trigger (`bejelentkezik`) and outcome but not the precondition (which subscription level), so the card supplies it as `Amennyiben` — the same gap `pte-openspec-bdd-tests` fills for `Given`.
- **`BDD teszt` left as a placeholder** — this skill writes no Gherkin; it authors the empty `BDD teszt` section that `pte-openspec-bdd-tests` fills in place from the same Scenarios. The story card is the smallest unit and the home of its own BDD test.
- **Placeholders, not guesses** — priority, estimate, DoR and DoD are refinement inputs; the skill leaves them for the team rather than inventing values.
- **Trace row reprinted verbatim** — the card carries the epic's map row unchanged, so the epic, this card, and the Gherkin embedded in the card all key off the same `STORY-…` ID.
