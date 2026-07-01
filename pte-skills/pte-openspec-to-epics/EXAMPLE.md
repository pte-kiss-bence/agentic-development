# Worked example: OpenSpec spec → Agile Epic

One end-to-end transformation showing the non-obvious moves: grouping Requirements into a single lean epic, writing the hypothesis, cutting one Requirement into two vertical-slice stories, assigning requirement-anchored trace IDs (with an aspect-slug on the split), and emitting the traceability map the downstream skills consume.

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

### Requirement: Előfizetés kezelése
A rendszer SHALL lehetővé tegye az előfizetés indítását és lemondását.

#### Scenario: Előfizetés indítása
- **WHEN** a felhasználó fizetős előfizetést indít
- **THEN** a rendszer aktiválja a fizetős hozzáférést

#### Scenario: Előfizetés lemondása
- **WHEN** a felhasználó lemondja az előfizetését
- **THEN** a rendszer a periódus végén visszaállítja az ingyenes szintet
```

## Output — Agile Epic file

`openspec/backlog/epics/elofizetes-cikkhozzaferes.md`

```markdown
# Cím
Előfizetés-alapú cikkhozzáférés — `EPIC-elofizetes-cikkhozzaferes`

## Háttér és kontextus
Az olvasók előfizetési szintje dönti el, mely cikkeket érhetik el. A szint
indítható és lemondható, a hozzáférés ehhez igazodik.

## Probléma / lehetőség
Ma nincs szinthez kötött hozzáférés-szabályozás; a fizetős tartalom nem
védett, és nincs önkiszolgáló előfizetés-kezelés.

## Hipotézis
Ha az előfizetési szinthez kötjük a cikkhozzáférést és önkiszolgáló
előfizetés-kezelést adunk, akkor a fizető olvasók aránya nő a regisztrált
olvasók körében, mérve a havi fizetős konverzióval.

## Hatókör és nem-célok
- Hatókör: szint szerinti hozzáférés, előfizetés indítása, lemondása.
- Nem-cél: fizetési szolgáltató integrációja, árképzés, próbaidőszak.

## Érintettek
Olvasó, szerkesztőség, számlázás.

## Sikermutatók
- Fizetős konverzió (havi) ≥ X%.
- Jogosulatlan fizetős-cikk hozzáférés = 0.

## Magas szintű elfogadási kritériumok
- A megjelenített cikkek köre mindig az aktuális előfizetési szintet tükrözi.
- Indítás után azonnal, lemondás után a periódus végén vált a szint.

## User story-k
- `STORY-elofizetes-cikkhozzaferes-elofizetesi-szint-szerinti-cikkhozzaferes`
  Mint olvasó, szeretném a szintemnek megfelelő cikkeket látni, hogy csak a
  jogosult tartalomhoz férjek hozzá.
  - Ingyenes előfizető az ingyenes cikket látja, a fizetőset nem.
  - Fizető előfizető a fizetős cikket is látja.
- `STORY-elofizetes-cikkhozzaferes-elofizetes-kezelese-inditas`
  Mint olvasó, szeretnék fizetős előfizetést indítani, hogy elérjem a fizetős
  cikkeket.
  - Indítás után a fizetős hozzáférés aktív.
- `STORY-elofizetes-cikkhozzaferes-elofizetes-kezelese-lemondas`
  Mint olvasó, szeretném lemondani az előfizetésem, hogy ne fizessek tovább.
  - Lemondás után a periódus végén visszaáll az ingyenes szint.

## Függőségek és kockázatok
- Függ a hitelesítéstől (bejelentkezett felhasználó).
- Kockázat: a lemondás időzítése (azonnali vs. periódus végi) félreérthető.

## Forrás-spec hivatkozás

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| `STORY-elofizetes-cikkhozzaferes-elofizetesi-szint-szerinti-cikkhozzaferes` | `EPIC-elofizetes-cikkhozzaferes` | Előfizetési szint szerinti cikkhozzáférés | Ingyenes előfizető csak ingyenes cikket lát; Fizető előfizető a fizetős cikket is látja |
| `STORY-elofizetes-cikkhozzaferes-elofizetes-kezelese-inditas` | `EPIC-elofizetes-cikkhozzaferes` | Előfizetés kezelése | Előfizetés indítása |
| `STORY-elofizetes-cikkhozzaferes-elofizetes-kezelese-lemondas` | `EPIC-elofizetes-cikkhozzaferes` | Előfizetés kezelése | Előfizetés lemondása |
```

## Why each move

- **One lean epic, two Requirements** — both Requirements serve a single user value (subscription-based access), so they group into one epic rather than one-epic-per-Requirement. Well under the ~10-story ceiling.
- **Hypothesis** — `Ha … akkor … mérve …` ties the epic to a measurable outcome, not a task list.
- **Vertical-slice split** — "Előfizetés kezelése" cuts into *indítás* and *lemondás*: two slices by user value, each independently shippable — not a "backend" / "frontend" layer split.
- **Requirement-anchored IDs** — `STORY-<epic-slug>-<requirement-slug>` stays stable if stories are reordered. Where one Requirement splits, the `-<aspect-slug>` (`inditas` / `lemondas`) disambiguates without a positional number.
- **Traceability map** — the `Forrás-spec hivatkozás` table keeps the `#### Scenario` titles **verbatim**. That is the contract: `pte-openspec-to-stories` carries each row into its story card, then `pte-openspec-bdd-tests` reads that row, tags the card's embedded Gherkin `Scenario` with its `@STORY-…` by matching the title, and still sources the step text from the spec.

---

## Reconcile example: change delta → existing epic

A follow-up change request extends the same capability. The epic above already exists, so this skill runs in **reconcile** mode: it resolves that epic, reuses `EPIC-elofizetes-cikkhozzaferes` **verbatim**, and folds the delta in — it does **not** mint a new epic.

### Input — the change delta

`openspec/changes/cikk-elonezet/specs/cikk-hozzaferes/spec.md`

```markdown
## ADDED Requirements

### Requirement: Fizetős cikk előnézete
A rendszer SHALL az ingyenes előfizetőnek rövid előnézetet mutasson a fizetős cikkből.

#### Scenario: Ingyenes előfizető előnézetet lát
- **WHEN** ingyenes előfizetésű felhasználó fizetős cikket nyit meg
- **THEN** a rendszer megjeleníti a cikk első bekezdését előnézetként
- **THEN** a rendszer előfizetésre ösztönző felhívást mutat

## MODIFIED Requirements

### Requirement: Előfizetési szint szerinti cikkhozzáférés
A rendszer SHALL az előfizetés szintje szerint engedélyezze a cikkekhez való hozzáférést.

#### Scenario: Ingyenes előfizető csak ingyenes cikket lát
- **WHEN** ingyenes előfizetésű felhasználó bejelentkezik
- **THEN** a rendszer megjeleníti az ingyenes cikket
- **THEN** a rendszer a fizetős cikket előnézettel jelzi
```

### Resolve the mode

Scan `openspec/backlog/epics/` for an epic whose *Forrás-spec hivatkozás* rows trace to `cikk-hozzaferes` → one hit, `EPIC-elofizetes-cikkhozzaferes` → **reconcile** it (no `AskUserQuestion` needed; a single candidate).

### Output — the epic, edited in place

- **New story** for the ADDED requirement, minted under the existing epic:
  `STORY-elofizetes-cikkhozzaferes-fizetos-cikk-elonezete` — *Mint ingyenes olvasó, szeretnék előnézetet látni a fizetős cikkből, hogy eldönthessem, előfizetek-e.*
- **Extended coverage** for the MODIFIED scenario: `Ingyenes előfizető csak ingyenes cikket lát` already belongs to `STORY-…-elofizetesi-szint-szerinti-cikkhozzaferes`; its map row + acceptance criteria pick up the changed `THEN` (előnézettel jelzi) — **no new story minted for it**.
- Everything else (hypothesis, scope, the *indítás* / *lemondás* stories) stays byte-for-byte.

The *Forrás-spec hivatkozás* map gains exactly one row; one existing row is touched:

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| `STORY-elofizetes-cikkhozzaferes-elofizetesi-szint-szerinti-cikkhozzaferes` | `EPIC-elofizetes-cikkhozzaferes` | Előfizetési szint szerinti cikkhozzáférés | Ingyenes előfizető csak ingyenes cikket lát *(módosított)*; Fizető előfizető a fizetős cikket is látja |
| `STORY-elofizetes-cikkhozzaferes-fizetos-cikk-elonezete` *(új)* | `EPIC-elofizetes-cikkhozzaferes` | Fizetős cikk előnézete | Ingyenes előfizető előnézetet lát |

### Why each move

- **EPIC reused verbatim** — reconcile never re-mints an epic ID; the epic file is edited, not regenerated.
- **ADDED → new `STORY-…`** — a new requirement gets a new requirement-anchored story under the same epic.
- **MODIFIED → extend, don't duplicate** — the scenario already maps to a story, so its coverage/AC update in place; minting a second story for it would duplicate the same behaviour.
- **Diff-don't-clobber is load-bearing** — untouched stories and sections are left exactly as the team groomed them; only the delta's footprint changes.
- **Where mint would be wrong here** — the capability already has an epic; minting a second would fork the backlog and split the trace IDs. Reconcile keeps one epic as the home of the capability.
