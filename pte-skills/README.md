# pte-skills — OpenSpec → groomed backlog pipeline

Ez a mappa a `pte-openspec-*` skillcsaládot tartalmazza: egy **lineáris láncot** (pipeline), amely az [OpenSpec](https://github.com/fission-ai/openspec) specifikációkat egy csiszolt, tesztelhető Agile backlogra fordítja le. Nem alkalmazáskód — Claude Code skillek, amelyek egymásra épülnek, szigorúan sorrendben.

A lánc **két artefaktum-fajtát** termel:

- **Epic** — a legnagyobb egység, amivel dolgozunk. A leírásában tartalmazza a hozzá tartozó story-kat (*User story-k* szakasz).
- **Story kártya** — a legkisebb egység, amivel dolgozunk. A leírásában tartalmazza a saját BDD tesztjét/tesztjeit (`BDD teszt` szakasz, beágyazott Gherkin).

**Nincs külön `features/` fa** — a BDD tesztek a story kártyákba ágyazódnak, így egy story önmagában teljes, tesztelhető munkaegység.

A skillek prózája (`SKILL.md`) angol, de **minden generált artefaktum tartalma magyar** — összhangban az `openspec/config.yaml` artefaktum-konvenciójával. A közös szabályok egy helyen élnek: [`pte-openspec-shared/CONVENTIONS.md`](pte-openspec-shared/CONVENTIONS.md).

## A workflow áttekintése

```
                 openspec explore / propose
                            │
                            ▼
              openspec/specs/**/spec.md
              (### Requirement → #### Scenario → WHEN/THEN)
                            │
                            ▼
             ┌──────────────────────────────┐
             │     pte-openspec-to-epics     │  MINTS: EPIC-… / STORY-… ID-k
             │            → epics/           │         + Forrás-spec hivatkozás map
             │  (epic tartalmazza story-kat) │
             └──────────────────────────────┘
                            │  CONSUMES: epic map + ID-k
                            ▼
             ┌──────────────────────────────┐
             │    pte-openspec-to-stories    │  egy kártya / STORY-…
             │           → stories/          │  BDD teszt szakasz = placeholder
             └──────────────────────────────┘
                            │  CONSUMES: story kártyák
                            ▼
             ┌──────────────────────────────┐
             │     pte-openspec-bdd-tests    │  a kártya BDD teszt szakaszát
             │  (Gherkin a story kártyába,   │  helyben tölti ki (nincs új fájl)
             │   helyben — nincs features/)  │
             └──────────────────────────────┘
```

A `pte-openspec` skill a lánc **routere**: megmondja, melyik skill mit csinál és milyen sorrendben.

## A lánc lépései

A lépések **szigorúan sorrendben** futnak — mindegyik az előző által termelt artefaktumot fogyasztja. A 4. függ a 3.-tól, az a 2.-tól.

### 0. Explore / propose (OpenSpec)

Az `openspec-explore` / `openspec-propose` (vagy az `opsx:*` parancsok) egy ötletből specifikációt vagy change deltát készítenek az `openspec/` alá. Minden `### Requirement` `#### Scenario` blokkokat tartalmaz, ezek pedig `WHEN`/`THEN` felsorolásokat — ez a viselkedés forrása az egész lánc számára.

### 1. `pte-openspec-to-epics` → `epics/`

A specifikáció részletes viselkedését **lean Agile epikekre** göngyöli fel — epikenként egy markdown fájl. Az epic érték-orientált stratégiai konténer: hipotézist fogalmaz meg, kijelöli a hatókört és a nem-célokat, és **vertikális szeletekre** (user value / journey mentén, nem technikai réteg szerint) vágott user story-kra bomlik. Az epic *User story-k* szakasza **tartalmazza a hozzá tartozó story-kat**.

Ez a skill **MINTS** — vagyis ő hozza létre — a lánc trace ID-jait:

- `EPIC-<epic-slug>` — epikenként egy.
- `STORY-<epic-slug>-<requirement-slug>` — story-nként egy; ha több story osztozik egy requirementen, stabil `-<aspect-slug>` toldalék kerül rá (soha nem sorszám).

És emittálja a **Forrás-spec hivatkozás** táblát — ez a lefelé irányuló skillek szerződése:

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|

A requirement→story felosztás **ítéleti döntés**, nem vezethető le pusztán a specből — ezért létezik a map, és ezért olvassák a downstream skillek, nem pedig újraszármaztatják.

### 2. `pte-openspec-to-stories` → `stories/`

Az epic minden story-ját teljes, **refinement-ready** Agile story kártyává bővíti — story-nként egy fájl. Elsődleges bemenete **az epic fájl**, nem a nyers spec. A skill **CONSUMES** — az epic által mintázott `STORY-…` / `EPIC-…` ID-kat szó szerint újrahasználja, soha nem mint újat. Story-nként pontosan egy kártya.

A kártya a **3 C**-t testesíti meg (Card, Conversation, Confirmation). Elfogadási kritériumai 1:1-ben a map által a story-hoz rendelt `#### Scenario`-k `WHEN`/`THEN` bulletjeiből bővülnek (`Amennyiben`/`Amikor`/`Akkor`), a hiányzó előfeltételt (`Amennyiben`) a skill pótolja. Tartalmaz INVEST-ellenőrzést, valamint DoR/DoD/prioritás/becslés placeholdereket a csapatnak.

A kártya egy **`BDD teszt`** szakaszt is kap — **placeholderként**. Ez a skill ide nem ír Gherkint; a szakaszt a következő lépés tölti ki.

### 3. `pte-openspec-bdd-tests` → beágyazva a story kártyába

A lánc **utolsó lépése**. Bemenete a **story kártyák** (`stories/`), nem a nyers spec. Minden kártya *Forrás-hivatkozás* sora megmondja, mely `#### Scenario`-kat fedi le és mely trace ID-kkal kell taggelni. A skill ezekből a scenariókból **deklaratív Gherkint** ír, és a kártya `BDD teszt` szakaszát **helyben** tölti ki — **nem hoz létre külön fájlt**, és a kártya többi szakaszához nem nyúl.

A vezérszó a `declarative`: minden lépés azt írja le, *mit* csinál a rendszer, nem azt, *hogyan* kattint a felhasználó (nincs UI-, route- vagy mezőszintű részlet). Leképezés majdnem 1:1: `### Requirement` → `Szabály:`, `#### Scenario` → `Forgatókönyv:` (vagy `Forgatókönyv vázlat:` ha csak adat változik), `WHEN` → `Amikor`, `THEN` → `Akkor`. Az egyetlen rés a `Given`/`Adott`: a spec az előfeltételt ritkán mondja ki, ezt a skill pótolja. Magyar spec esetén a beágyazott blokk első sora `# language: hu`.

A beágyazott Gherkin a kártya saját `@EPIC-…` / `@STORY-…` tagjeit viseli (a *Forrás-hivatkozás* sorból, szó szerint) — így epic, story kártya és a benne lévő Gherkin egy-az-egyben illeszkedik. A lépések szövege mindig a specből származik, soha nem a kártya prózájából.

## Trace ID-k és a szerződés

Minden 2. lépés utáni stage ugyanarra a trace ID-ra kulcsol, ezért egy epic, a story kártyái és a kártyákba ágyazott Gherkin egy-az-egyben összeérnek. A lánc lineáris: a 4. a 3.-at fogyasztja, az a 2.-at.

**Ownership:** `pte-openspec-to-epics` **MINTS**; `pte-openspec-to-stories` és `pte-openspec-bdd-tests` **CONSUME** (szó szerint hivatkoznak, soha nem mintáznak újra).

## Közös konvenciók

A [`pte-openspec-shared/CONVENTIONS.md`](pte-openspec-shared/CONVENTIONS.md) az egyetlen igazságforrás. A lényeg:

- **Output nyelv** — az artefaktum tartalma magyar; **szó szerint marad** (nem fordul): kód, azonosítók, API-nevek, CLI-parancsok, fájlútvonalak, `### Requirement` nevek, `#### Scenario` címek, `EPIC-…`/`STORY-…` ID-k és slugok, commit-típus kulcsszavak (`feat`/`fix`/…).
- **Forrás-spec feloldás** — change delta (`openspec list --json` → change → `openspec/changes/<id>/specs/**/spec.md`) vagy fő spec (`openspec/specs/<capability>/spec.md`). Store megnevezésekor `--store <id>`. Homályos bemenetnél a skill **AskUserQuestion**-nel listázza az opciókat.
- **Diff, ne clobber** — felülírás előtt diff; kézzel szerkesztett tartalmat sosem írunk felül. Az epics és stories skill fájlonként egy artefaktumot ír; a bdd-tests skill semmi újat nem ír — a meglévő story kártyák `BDD teszt` szakaszát tölti ki helyben, a többihez nem nyúl.

## Mappastruktúra

```
pte-skills/
├─ README.md                       ← ez a fájl
├─ pte-openspec/SKILL.md           ← router (a lánc térképe)
├─ pte-openspec-shared/
│  └─ CONVENTIONS.md               ← közös szabályok (egy hely)
├─ pte-openspec-to-epics/
│  ├─ SKILL.md
│  └─ EXAMPLE.md                   ← teljes kidolgozott transzformáció
├─ pte-openspec-to-stories/
│  ├─ SKILL.md                     ← story kártya = legkisebb egység, BDD teszt szakasszal
│  └─ EXAMPLE.md
└─ pte-openspec-bdd-tests/
   ├─ SKILL.md                     ← Gherkint a story kártyába ágyazza (nincs features/)
   ├─ BDD-RULES.md                 ← Gherkin szabályok + anti-pattern-ek
   └─ EXAMPLE.md
```

Generált artefaktumok (nem ebben a mappában — a projekt gyökeréből): `epics/` az epikek, `stories/` a story kártyák (a beágyazott BDD tesztekkel együtt). Külön `features/` fa **nincs**.

Minden skillhez tartozik egy `EXAMPLE.md` a teljes kidolgozott átalakítással (be → ki, ID-kkal és map-pel). A `pte-openspec-bdd-tests` `BDD-RULES.md`-je a deklaratív Gherkin szabálykészletét és anti-pattern-jeit tartalmazza — a skill lépéslépés előtt betölti.
