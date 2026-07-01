# Worked example: Agile Epic story → refinement-ready story card

Two end-to-end transformations showing the non-obvious moves of the **lean** story-card schema — English `##` headings, Hungarian body. The card has five visible sections (`## Description` / `## Context` / `## BDD Test` / `## Estimation` / `## Risks and Dependencies`); the `Mint …` line lives **inside** `## Description` (no separate *User story* heading), and the acceptance criteria are **not** a section here — they are the declarative Gherkin that `pte-openspec-bdd-tests` later fills into the `## BDD Test` placeholder.

The two demos cover both modes:
- **Demo 1 (epic-driven)** — expand one story from an epic's *User story-k* list, reusing its `STORY-…`/`EPIC-…` IDs and `Mint …` line verbatim. The card carries **no** trace row: the map lives in the epic.
- **Demo 2 (epic-less)** — author a standalone card straight from a small change's delta, minting a capability-namespaced `STORY-…`, leaving the parent `EPIC-…` empty, and **self-carrying** the map inside a pipeline-only fence.

---

## Demo 1 (epic-driven): epic story → story card

### Input — one story from the epic

From `openspec/backlog/epics/EPIC-belso-hozzaferes.md` (produced by `pte-openspec-to-epics`), the `## User story-k` entry for this story:

```markdown
## User story-k
- `STORY-belso-hozzaferes-m365-szerepkorok`
  Mint belső felhasználó, szeretném M365-tel belépni és a szerepkörömnek
  megfelelő jogot kapni, hogy csak az admin szerkeszthessen, a reader pedig
  biztonságosan csak olvasson.
```

Its *Forrás-spec hivatkozás* row — held in the **epic's** pipeline-only fence, not in the card:

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| STORY-belso-hozzaferes-m365-szerepkorok | EPIC-belso-hozzaferes | M365 alapú belső hozzáférés és szerepkörök | Admin szerkeszthet; Reader csak olvas |

The card reads the requirement→story→scenario split from **this epic row** (there is no card trace row in epic-driven mode). The two `#### Scenario`s named in the row, traced to `openspec/specs/belso-hozzaferes/spec.md`, are the behavioural source the future BDD test keys off:

```markdown
#### Scenario: Admin szerkeszthet
- **WHEN** `admin` szerepkörű felhasználó szerkesztő műveletet indít
- **THEN** a rendszer engedélyezi a műveletet

#### Scenario: Reader csak olvas
- **WHEN** `reader` szerepkörű felhasználó szerkesztő műveletet indít
- **THEN** a rendszer elutasítja a műveletet
- **THEN** a rendszer csak olvasási hozzáférést enged
```

### Output — story card file

`openspec/backlog/stories/belso-hozzaferes/STORY-belso-hozzaferes-m365-szerepkorok.md`

```markdown
# M365 belépés és szerepkörök · STORY-belso-hozzaferes-m365-szerepkorok

Parent epic: `EPIC-belso-hozzaferes`

## Description
Mint **belső felhasználó**, szeretném **M365-tel belépni és a szerepkörömnek megfelelő jogot kapni**, hogy **csak az admin szerkeszthessen, a reader pedig biztonságosan csak olvasson**.

Ez a story a belső hozzáférés alapja: Entra ID (M365 / OIDC) hitelesítés, két szerepkörrel — `admin` (szerkeszt) és `reader` (csak olvas).

## Context
A szerepkör-szeparáció védi a szerkesztő műveleteket (feltöltés, override, lezárás).

## BDD Test
_(kitölti a `pte-openspec-bdd-tests`)_

## Estimation
- **3-Points becslés (ideális óra):** O 4 / M 8 / P 16 | **Eβ 9**, σ 2
- **Becsült munkaóra:** 9 ó (≈ 1,5 ideális nap)
- **Story Point:** 5

## Risks and Dependencies
- Függőség: Entra ID (M365/OIDC) tenant és szerepkör-hozzárendelés.
- Kapcsolódás: a szerkesztő műveletek (`EPIC-excel-ingest`, admin override, lezárás) erre a szerepkör-ellenőrzésre épülnek.
- Kockázat: téves szerepkör-hozzárendelés → jogosulatlan szerkesztés.
```

### Why each move

- **Epic is the input, not the spec** — the story set and IDs already exist; this skill only expands. The `STORY-…`/`EPIC-…` IDs and the `Mint …` line are reused **verbatim** — nothing minted here (see `CONVENTIONS.md`).
- **The `Mint …` line opens `## Description`** — there is no separate *User story* heading; the user-value statement leads the description, then one narrative sentence.
- **No trace row on the card** — the requirement→story→scenario map lives in the epic's pipeline-only fence; the card keys off the epic row. This skill reads that row to trace the `Admin szerkeszthet` / `Reader csak olvas` `WHEN`/`THEN` back to `openspec/specs/belso-hozzaferes/spec.md`, so `pte-openspec-bdd-tests` can turn those scenarios into the card's BDD test — the story's acceptance criteria.
- **`## BDD Test` left as a placeholder** — this skill writes no Gherkin; it authors the empty placeholder that `pte-openspec-bdd-tests` fills in place from the same scenarios.

---

## Demo 2 (epic-less): small change delta → standalone story card

A small change request warrants a story but no epic, so it **skips `pte-openspec-to-epics`** and runs this skill in **epic-less** mode: the card is authored straight from the delta, the `STORY-…` is minted here (capability-namespaced on `ertesites-sablon`), the parent `EPIC-…` line is left empty, and the card **self-carries** its map inside a pipeline-only fence.

### Input — the change delta

`openspec/changes/ertesites-sablon-szerkesztes/specs/ertesites-sablon/spec.md`

```markdown
## ADDED Requirements

### Requirement: Sablon szerkesztése
A rendszer SHALL engedje az adminnak az értesítő e-mail sablon szövegét
szerkeszteni és menteni, és SHALL utasítsa el az érvénytelen sablont.

#### Scenario: Sablon mentése
- **WHEN** admin érvényes sablonszöveget ment
- **THEN** a rendszer elmenti a sablont
- **THEN** a rendszer a kiküldött üzenetekben az új szöveget használja

#### Scenario: Érvénytelen sablon elutasítása
- **WHEN** admin érvénytelen sablont ment (pl. hibás helyőrző)
- **THEN** a rendszer elutasítja a mentést
- **THEN** a rendszer hibaüzenetet mutat
```

No epic exists for the `ertesites-sablon` capability, and the change is a single small item → **epic-less mode**. (Had `ertesites-sablon` already had an epic, this would instead be `pte-openspec-to-epics` in **reconcile** mode — never an orphaned story alongside an existing epic.)

### Output — story card file

`openspec/backlog/stories/ertesites-sablon/STORY-ertesites-sablon-sablon-szerkesztes.md`

```markdown
# Értesítő e-mail sablon karbantartása · STORY-ertesites-sablon-sablon-szerkesztes

Parent epic: _(nincs — epic nélküli story)_

## Description
Mint **admin**, szeretném **az értesítő e-mail sablon szövegét szerkeszteni**, hogy **a kiküldött üzenetek naprakészek legyenek**.

A sablon mentése azonnal hat a következő kiküldött üzenetekre; az érvénytelen sablont a rendszer elutasítja.

## Context
Kis, önálló változtatás az értesítő e-mail kézbesítésen; nincs hozzá epic.

## BDD Test
_(kitölti a `pte-openspec-bdd-tests`)_

## Estimation
- **3-Points becslés (ideális óra):** O 2 / M 4 / P 8 | **Eβ 4**, σ 1
- **Becsült munkaóra:** 4 ó (≈ 0,7 ideális nap)
- **Story Point:** 2

## Risks and Dependencies
- Függőség: az értesítő e-mail kiküldő pipeline, amely a sablonszöveget használja.
- Kockázat: érvénytelen helyőrző a sablonban → hibás vagy nem kézbesített e-mail.

<!-- pipeline-only:start -->
## Forrás-hivatkozás
| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|
| STORY-ertesites-sablon-sablon-szerkesztes | (üres) | Sablon szerkesztése | Sablon mentése; Érvénytelen sablon elutasítása |
<!-- pipeline-only:end -->
```

### Why each move

- **Delta is the input, not an epic** — with no epic map to consume, the skill reads the delta's Requirement/Scenario directly and cuts the story itself (the split judgement `to-epics` normally owns, scoped to this one small change).
- **Capability-namespaced ID, minted here** — `STORY-ertesites-sablon-sablon-szerkesztes` uses the capability slug, not an epic slug. It is **stable, not provisional** (see `CONVENTIONS.md`) — this is a first-class mode, not a degraded fallback.
- **Empty parent `EPIC-…`** — the marker that this is an epic-less story; `pte-openspec-jira-sync` parents it under the standalone collector epic.
- **Self-carried map inside a pipeline-only fence** — with no epic to hold the *Forrás-hivatkozás* row, the card carries its own row (hidden from Jira by the fence) so `pte-openspec-bdd-tests` knows which `#### Scenario`s to turn into the card's BDD test.

---

## Why these moves

- **Lean schema, five visible sections** — a card shows `## Description`, `## Context`, `## BDD Test`, `## Estimation`, and `## Risks and Dependencies`. English headings, Hungarian body (per `CONVENTIONS.md`).
- **`## Estimation` is the reference base** — three stacked top-level bullets: the **`3-Points becslés`** (`O`/`M`/`P`/`Eβ`/`σ` inline on one line, side by side), then **`Becsült munkaóra`** (`Eβ` hours + ideal-day equivalent), then **`Story Point`** (`<SP>`, a bare integer derived from `Eβ` by the fixed table in `ESTIMATION.md`). It is AI-authored and authoritative in the pipeline; managers layer their internal/client multipliers on it downstream (out of scope here). A story that trips `SP ≥ 13` or `σ/Eβ > 0.5` gets a `⚠` split-warning line but still ships — Demo 1's 5 SP / low σ is well clear of it. The epic rollup drops the `3-Points` bullet (O/M/P don't sum) and shows only `Becsült munkaóra` + `Story Point`; the exact rendered format lives in `ESTIMATION.md`.
- **The `Mint …` line lives inside `## Description`** — there is no separate *User story* heading; the user-value statement opens the description, then one or two narrative sentences.
- **`## BDD Test` is a placeholder, filled later, and IS the acceptance criteria** — this skill writes no Gherkin; `pte-openspec-bdd-tests` fills the placeholder from the mapped `#### Scenario`s, and that Gherkin **is** the story's acceptance criteria. The old *sections* are dropped: *Elfogadási kritériumok* (now the Gherkin), *INVEST-ellenőrzés*, *Készenléti feltétel (DoR)*, *Elkészültségi feltétel (DoD)*, *Prioritás*. (*Becslés* is **not** dropped — it returns as `## Estimation`, reshaped into the PERT + Story-Points reference base.)
- **INVEST stays as a rule, not a section** — both cards above satisfy INVEST (Independent, Negotiable, Valuable, Estimable, Small, Testable) even though no `INVEST-ellenőrzés` section is printed. Conformance is a hard gate: a story that can't be made INVEST-conform is re-split, not shipped.
- **Trace lives where the map lives** — an epic-driven card carries **no** trace row (the *Forrás-spec hivatkozás* map stays in the epic); an epic-less card **self-carries** its *Forrás-hivatkozás* row inside a `pipeline-only` fence, since there is no epic to hold it.
