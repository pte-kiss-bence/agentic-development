# Worked example: Agile Epic story → refinement-ready story card

One end-to-end transformation showing the non-obvious moves: taking a single story out of an epic's *User story-k* list, reusing its `STORY-…`/`EPIC-…` IDs and `Mint …` line verbatim, tracing the map's `#### Scenario`s back to the spec, and expanding their `WHEN`/`THEN` into `Amennyiben`/`Amikor`/`Akkor` acceptance criteria with a supplied precondition.

## Input — one story from the epic

From `openspec/backlog/epics/EPIC-elofizetes-cikkhozzaferes.md` (produced by `pte-openspec-to-epics`):

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

`openspec/backlog/stories/elofizetes-cikkhozzaferes/elofizetesi-szint-szerinti-cikkhozzaferes.md`

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

---

## Epic-less example: small change delta → standalone story card

A small change request warrants a story but no epic, so it **skips `pte-openspec-to-epics`** and runs this skill in **epic-less** mode: the card is authored straight from the delta, the `STORY-…` is minted here (capability-namespaced), and the parent `EPIC-…` line is left empty.

### Input — the change delta

`openspec/changes/hirlevel-leiratkozas/specs/hirlevel/spec.md`

```markdown
## ADDED Requirements

### Requirement: Egykattintásos leiratkozás
A rendszer SHALL a hírlevél láblécében egykattintásos leiratkozást kínáljon.

#### Scenario: Feliratkozott felhasználó leiratkozik
- **WHEN** a felhasználó a hírlevél leiratkozó linkjére kattint
- **THEN** a rendszer azonnal leiratkoztatja
- **THEN** a rendszer megerősítő oldalt mutat
```

No epic exists for the `hirlevel` capability, and the change is a single small item → **epic-less mode**. (Had `hirlevel` already had an epic, this would instead be `pte-openspec-to-epics` in **reconcile** mode — never an orphaned story alongside an existing epic.)

### Output — story card file

`openspec/backlog/stories/hirlevel/egykattintasos-leiratkozas.md`

```markdown
# Cím
Egykattintásos leiratkozás — `STORY-hirlevel-egykattintasos-leiratkozas`
(epic nélküli — `EPIC-` üres)

## User story
Mint hírlevél-feliratkozó, szeretnék egy kattintással leiratkozni, hogy
gyorsan megszüntethessem a feliratkozásom.

## Kontextus
Kis, önálló változtatás a hírlevél-kézbesítésen; nincs hozzá epic.

## Elfogadási kritériumok
- Amennyiben a felhasználó feliratkozott a hírlevélre,
  Amikor a leiratkozó linkre kattint,
  Akkor a rendszer azonnal leiratkoztatja,
  És megerősítő oldalt mutat.

## BDD teszt
_(kitölti a `pte-openspec-bdd-tests`)_

## INVEST-ellenőrzés
Independent ✓ · Negotiable ✓ · Valuable ✓ · Estimable ✓ · Small ✓ · Testable ✓
— önállóan szállítható, egyetlen viselkedés.

## Készenléti feltétel (DoR)
_(csapat tölti ki refinementen)_

## Elkészültségi feltétel (DoD)
_(csapat tölti ki refinementen)_

## Prioritás
_(csapat tölti ki)_

## Becslés
_(story point — refinementen becsülve)_

## Függőségek és kockázatok
- Nincs testvér-történet (önálló change).

## Forrás-hivatkozás
| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| `STORY-hirlevel-egykattintasos-leiratkozas` | _(nincs)_ | Egykattintásos leiratkozás | Feliratkozott felhasználó leiratkozik |
```

### Why each move

- **Delta is the input, not an epic** — with no epic map to consume, the skill reads the delta's Requirement/Scenario directly and cuts the story itself (the split judgement `to-epics` normally owns, scoped to this one small change).
- **Capability-namespaced ID, minted here** — `STORY-hirlevel-egykattintasos-leiratkozas` uses the capability slug, not an epic slug. It is **stable, not provisional** (see `CONVENTIONS.md`) — this is a first-class mode, not a degraded fallback.
- **Empty parent `EPIC-…`** — the marker that this is an epic-less story; `pte-openspec-jira-sync` parents it under the standalone collector epic (`trace:EPIC-standalone`).
- **Everything else identical** — anatomy, the 1:1 Scenario→AC trace, the supplied `Amennyiben`, the `BDD teszt` placeholder and diff-don't-clobber all match epic-driven mode exactly.
- **When to escalate** — if this had grown to several stories or wanted strategic framing, that's the signal to stop and run `to-epics` (mint or reconcile) instead of minting many epic-less cards.
