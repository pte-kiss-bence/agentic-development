# pte-skills — OpenSpec → groomed backlog pipeline

Ez a mappa a `pte-openspec-*` skillcsaládot tartalmazza: egy **két fázisú** pipeline-t, amely az [OpenSpec](https://github.com/fission-ai/openspec) specifikációkat egy csiszolt, tesztelhető Agile backlogra fordítja le (**planning-lánc**), majd a change taskjait **teszt-first** kóddá implementálja (**build stage**). Nem alkalmazáskód — Claude Code skillek. A két fázis **nem táplálja egymást**: a planning-lánc lépései szigorú sorrendben egymásra épülnek, a build stage viszont az OpenSpec change taskjaiból dolgozik, függetlenül a planning-artefaktumoktól. A közös horgony mindkettőnél a **spec** — oda vezetnek vissza, nem egymáshoz.

A lánc **két artefaktum-fajtát** termel:

- **Epic** — a legnagyobb egység, amivel dolgozunk. A leírásában tartalmazza a hozzá tartozó story-kat (*User story-k* szakasz).
- **Story kártya** — a legkisebb egység, amivel dolgozunk. A leírásában tartalmazza a saját BDD tesztjét/tesztjeit (`BDD teszt` szakasz, beágyazott Gherkin).

**Nincs külön `features/` fa** — a BDD tesztek a story kártyákba ágyazódnak, így egy story önmagában teljes, tesztelhető munkaegység.

A skillek prózája (`SKILL.md`) angol, de **minden generált artefaktum tartalma magyar** — összhangban az `openspec/config.yaml` artefaktum-konvenciójával. A közös szabályok egy helyen élnek: [`pte-openspec-shared/CONVENTIONS.md`](pte-openspec-shared/CONVENTIONS.md).

## A workflow áttekintése

```
                 openspec explore / propose            (előfeltétel)
                            │
                            ▼
              openspec/specs/**/spec.md  ◀── közös horgony ──┐
              (### Requirement → #### Scenario → WHEN/THEN)   │
                            │                                 │
        ┌─── PLANNING ──────┘                                 │
        ▼                                                     │
   ┌──────────────────────────────┐                          │
   │  1. pte-openspec-to-epics     │  MINTS: EPIC-… / STORY-… │
   │            → epics/           │        + Forrás-spec map │
   └──────────────────────────────┘                          │
                  │  CONSUMES: epic map + ID-k                │
                  ▼                                           │
   ┌──────────────────────────────┐                          │
   │  2. pte-openspec-to-stories   │  egy kártya / STORY-…    │
   │           → stories/          │  BDD teszt = placeholder │
   └──────────────────────────────┘                          │
                  │  CONSUMES: story kártyák                  │
                  ▼                                           │
   ┌──────────────────────────────┐                          │
   │  3. pte-openspec-bdd-tests    │  Gherkin a kártyába,     │
   │  (helyben — nincs features/)  │  helyben (nincs új fájl) │
   └──────────────────────────────┘                          │
                  ╎ living doc                                │
                  ⇢ (tervezett) Playwright E2E — még nincs skill
                  │  CONSUMES: epics/ + stories/ (trace:… label)
                  ▼  (opcionális publish lépés)               │
   ┌──────────────────────────────┐                          │
   │  ✳ pte-openspec-jira-sync     │  Jira-ra tükröz          │
   │  (Atlassian MCP,              │  field-ownership,        │
   │   idempotens re-sync)         │  helyben ## Jira szinkron│
   └──────────────────────────────┘                          │
                                                              │
        ┌─── BUILD (külön ág, planning-től független) ────────┘
        ▼   CONSUMES: change tasks (nem a trace ID-k, nem epics/stories)
   ┌──────────────────────────────┐
   │  4. pte-openspec-tdd-apply    │  build stage: a change taskjait
   │   (red-green tdd taskonként,  │  teszt-first implementálja
   │    openspec-apply-change-en)  │  (user-invoked)
   └──────────────────────────────┘
```

> A diagramban az `epics/` és `stories/` a lemezes hely rövidítése: a valódi kimenet az **`openspec/backlog/epics/`** és **`openspec/backlog/stories/`** — a spec-ek mellett, az OpenSpec fán belül (de nem az OpenSpec CLI kezeli). Lásd *Output location* a [`CONVENTIONS.md`](pte-openspec-shared/CONVENTIONS.md)-ben.

A `pte-openspec` skill a lánc **routere**: megmondja, melyik skill mit csinál és milyen sorrendben.

## A change súlyához méretezve (right-sizing)

A planning-láncot **nem mindig futtatjuk végig** — azon a mélységen lépünk be, amit a change megkövetel. Egy change request tipikusan **létező viselkedést módosít**, ezért nem mindig kell új epic, sőt gyakran epic sem:

- **Teljes feature** → `pte-openspec-to-epics` **mint** módban (új capability, új epic), majd a teljes lánc.
- **Change egy létező epicbe** (change request default esete) → `pte-openspec-to-epics` **reconcile** módban: az érintett epic `EPIC-…`-ját szó szerint újrahasználja, és beleszövi a deltát (az új `STORY-…`-t minti, módosult scenariónál a meglévő story lefedettségét bővíti), diff-don't-clobber. Onnan `to-stories` → `bdd-tests` → `tdd-apply`.
- **Kis, önálló change** (story kell, epic nem) → `pte-openspec-to-stories` **epic-nélküli** módban: a kártyá(ka)t közvetlenül a deltából írja, a `to-epics`-et teljesen kihagyva. Az ID capability-névterű (`STORY-<capability-slug>-…`), stabil (nem provizórikus), a szülő `EPIC-…` sor üres.
- **Tweak / bugfix** → a planning-láncot teljesen kihagyva, egyből a build stage (`pte-openspec-tdd-apply`; bugnál előbb `diagnosing-bugs`, ha a gyökér-ok nem nyilvánvaló).

Mind a három planning-variáns ugyanabba a **`to-stories → bdd-tests → (tdd-apply) → jira-sync`** farokba fut össze. A mód-választás gépies: van-e már epic az érintett capabilityhez? Van → reconcile; nincs, de kell → mint; nincs és nem is kell → epic-nélküli story.

## A lánc lépései

A **planning-lánc** (1–3) lépései **szigorúan sorrendben** futnak — mindegyik az előző által termelt artefaktumot fogyasztja: a 3. a 2.-at, az a 1.-et. A **build stage** (4) ettől független ág — a change taskjaiból dolgozik, nem a planning-artefaktumokból.

### Előfeltétel — Explore / propose (OpenSpec)

Az `openspec-explore` / `openspec-propose` (vagy az `opsx:*` parancsok) egy ötletből specifikációt vagy change deltát készítenek az `openspec/` alá. Minden `### Requirement` `#### Scenario` blokkokat tartalmaz, ezek pedig `WHEN`/`THEN` felsorolásokat — ez a viselkedés forrása az egész pipeline számára. Ez **nem** `pte-openspec-*` skill, hanem a forrás, amit a pipeline olvas.

### 1. `pte-openspec-to-epics` → `openspec/backlog/epics/`

A specifikáció részletes viselkedését **lean Agile epikekre** göngyöli fel — epikenként egy markdown fájl. Az epic érték-orientált stratégiai konténer: hipotézist fogalmaz meg, kijelöli a hatókört és a nem-célokat, és **vertikális szeletekre** (user value / journey mentén, nem technikai réteg szerint) vágott user story-kra bomlik. Az epic *User story-k* szakasza **tartalmazza a hozzá tartozó story-kat**.

**Két mód** (a mód-választásról lásd a *right-sizing* szakaszt fentebb):
- **mint** — greenfield: új capability, nincs még epic → új epic(ek)et mint a nulláról (ez az eredeti viselkedés).
- **reconcile** — change delta, aminek a capabilityjéhez **már van** epic → nem mint új epicet: feloldja a meglévőt, az `EPIC-…`-ot szó szerint újrahasználja, és beleszövi a deltát (új `STORY-…` a hozzáadott requirementekhez, a meglévő story sorának bővítése módosult scenariónál), a nem érintett szakaszokat érintetlenül hagyva.

Ez a skill **MINTS** — vagyis ő hozza létre — a lánc trace ID-jait:

- `EPIC-<epic-slug>` — epikenként egy.
- `STORY-<epic-slug>-<requirement-slug>` — story-nként egy; ha több story osztozik egy requirementen, stabil `-<aspect-slug>` toldalék kerül rá (soha nem sorszám).

Reconcile módban az `EPIC-…`-ot **nem** mint újra, csak a delta új `STORY-…`-jait.

És emittálja a **Forrás-spec hivatkozás** táblát — ez a lefelé irányuló skillek szerződése:

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|

A requirement→story felosztás **ítéleti döntés**, nem vezethető le pusztán a specből — ezért létezik a map, és ezért olvassák a downstream skillek, nem pedig újraszármaztatják.

### 2. `pte-openspec-to-stories` → `openspec/backlog/stories/`

Az epic minden story-ját teljes, **refinement-ready** Agile story kártyává bővíti — story-nként egy fájl. Elsődleges bemenete **az epic fájl**, nem a nyers spec. A skill **CONSUMES** — az epic által mintázott `STORY-…` / `EPIC-…` ID-kat szó szerint újrahasználja, soha nem mint újat. Story-nként pontosan egy kártya.

**Epic-nélküli mód** (egyenrangú út, nem degradált fallback): kis, önálló change-nél, ami story-t érdemel de epicet nem, a kártyá(ka)t közvetlenül a delta specből írja, a `to-epics`-et kihagyva. Itt a skill **MINTS**: az ID capability-névterű (`STORY-<capability-slug>-…`), **stabil, nem provizórikus**, és a kártya szülő `EPIC-…` sora üres marad (ez a marker, hogy epic-nélküli). Ha az érintett capabilityhez **van** epic, ez a rossz mód — helyette `to-epics` reconcile.

A kártya a **3 C**-t testesíti meg (Card, Conversation, Confirmation). Elfogadási kritériumai 1:1-ben a map által a story-hoz rendelt `#### Scenario`-k `WHEN`/`THEN` bulletjeiből bővülnek (`Amennyiben`/`Amikor`/`Akkor`), a hiányzó előfeltételt (`Amennyiben`) a skill pótolja. Tartalmaz INVEST-ellenőrzést, valamint DoR/DoD/prioritás/becslés placeholdereket a csapatnak.

A kártya egy **`BDD teszt`** szakaszt is kap — **placeholderként**. Ez a skill ide nem ír Gherkint; a szakaszt a következő lépés tölti ki.

### 3. `pte-openspec-bdd-tests` → beágyazva a story kártyába

A **planning-lánc utolsó lépése**. Bemenete a **story kártyák** (`openspec/backlog/stories/`), nem a nyers spec. Minden kártya *Forrás-hivatkozás* sora megmondja, mely `#### Scenario`-kat fedi le és mely trace ID-kkal kell taggelni. A skill ezekből a scenariókból **deklaratív Gherkint** ír, és a kártya `BDD teszt` szakaszát **helyben** tölti ki — **nem hoz létre külön fájlt**, és a kártya többi szakaszához nem nyúl.

A vezérszó a `declarative`: minden lépés azt írja le, *mit* csinál a rendszer, nem azt, *hogyan* kattint a felhasználó (nincs UI-, route- vagy mezőszintű részlet). Leképezés majdnem 1:1: `### Requirement` → `Szabály:`, `#### Scenario` → `Forgatókönyv:` (vagy `Forgatókönyv vázlat:` ha csak adat változik), `WHEN` → `Amikor`, `THEN` → `Akkor`. Az egyetlen rés a `Given`/`Adott`: a spec az előfeltételt ritkán mondja ki, ezt a skill pótolja. Magyar spec esetén a beágyazott blokk első sora `# language: hu`.

A beágyazott Gherkin a kártya saját `@EPIC-…` / `@STORY-…` tagjeit viseli (a *Forrás-hivatkozás* sorból, szó szerint) — így epic, story kártya és a benne lévő Gherkin egy-az-egyben illeszkedik. A lépések szövege mindig a specből származik, soha nem a kártya prózájából.

A Gherkin **living documentation** — ember-olvasható viselkedésleírás, ebben a pipeline-ban **nem futtatható**. Egy **későbbi, külön fázis** implementálja **Playwright E2E** tesztként (erre még nincs skill). A deklaratív szabály ezért marad: a Gherkin UI-mentes; a Playwright/UI-részlet abba a jövőbeli E2E rétegbe kerül, sosem a Gherkinbe. Step-definition stubot ez a skill **nem** generál — a célfutó Playwright, nem Cucumber.

### ✳ `pte-openspec-jira-sync` → Jira-ra tükrözve (opcionális publish lépés)

A planning-lánc után futó **publish lépés**: a kész `openspec/backlog/epics/` és `openspec/backlog/stories/` artefaktumokat egy Jira projektbe tükrözi az **Atlassian MCP**-n keresztül. Bemenete az artefaktum-készlet és a trace ID-k (`trace:EPIC-…` / `trace:STORY-…` labelekként); nem mintáz újat, **CONSUME**-ol.

A vezérszó a **field-ownership**: a szinkron nem irány, hanem mezőnkénti **egy tulajdonos**. A *tartalmat* (Summary, Description, elfogadási kritériumok, beágyazott BDD, trace-label, epic↔story `parent`) a **local** birtokolja → Local → Jira (push). A *workflow-állapotot* (státusz, felelős, sprint, becslés, komment, Jira kulcs) a **Jira** birtokolja → Jira → Local, a kártya `## Jira szinkron` blokkjába pull-olva. Így az újrafuttatás idempotens, és egyik oldal munkáját sem írja felül.

Párosítás (idempotencia): rögzített `Jira kulcs` → `trace:…` label JQL-keresés → `createJiraIssue`. Epic előbb, story utána (a story `parent`-jéhez kell az epic kulcsa). Az **epic-nélküli story-k** (üres szülő `EPIC-…` sor) egy fenntartott **gyűjtő-epic** (`trace:EPIC-standalone`, pl. *Önálló változtatások*) alá kerülnek Jirában, hogy soha ne legyenek árvák. Alapból **dry-run** + megerősítés az első írás előtt; a státuszt sosem állítja (Jira-tulajdon). Előfeltétel: az Atlassian MCP bekötve és authentikálva.

### 4. `pte-openspec-tdd-apply` → tesztelt kód

A **build stage** — a tervezői lánc után fut, amikor a change implementálható. Bemenete nem a story kártyák, hanem az OpenSpec **change taskjai** (`tasks`), amelyeket az `openspec-apply-change` skillen keresztül olvas. Taskonként egy **red-green** hurkot futtat a `tdd` skillel: egy bukó teszt (**red**) → minimál kód, ami átmegy (**green**) → viselkedésenként ismételve; az első teszt a **tracer bullet**. Tiltott a horizontális rövidítés (összes teszt előre). A task csak akkor kap pipát (`- [ ]` → `- [x]`), ha minden viselkedésére van zöld teszt.

Ez a stage **nem a trace ID-kra kulcsol** és nem nyúl az `openspec/backlog/` artefaktumokhoz — a change taskjait fordítja tesztelt kóddá. **User-invoked** (`disable-model-invocation: true`): magától nem indul el, csak akkor, ha kézzel, névvel hívod — mert a build fázist szándékosan ember indítja, nem az agent autonóm módon.

## Trace ID-k és a szerződés

A **planning-lánc** minden lépése (1–3) ugyanarra a trace ID-ra kulcsol, ezért egy epic, a story kártyái és a kártyákba ágyazott Gherkin egy-az-egyben összeérnek. A build stage (4) **nem** kulcsol a trace ID-kra: az a change taskjaiból dolgozik, a planning-artefaktumokat nem olvassa. A két fázis csak a **specnél** találkozik.

**Ownership:** `pte-openspec-to-epics` **MINTS** az `EPIC-…`-ot és a gyerek `STORY-…`-kat (reconcile módban az `EPIC-…`-ot újrahasználja, csak az új `STORY-…`-t minti). `pte-openspec-to-stories` **CONSUME**-ol — kivéve **epic-nélküli módban**, ahol a saját `STORY-<capability-slug>-…`-ját minti. `pte-openspec-bdd-tests` csak **CONSUME**-ol (szó szerint hivatkozik, sosem mintáz újra).

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
├─ pte-openspec-bdd-tests/
│  ├─ SKILL.md                     ← Gherkint a story kártyába ágyazza (nincs features/)
│  ├─ BDD-RULES.md                 ← Gherkin szabályok + anti-pattern-ek
│  └─ EXAMPLE.md
├─ pte-openspec-jira-sync/
│  ├─ SKILL.md                     ← openspec/backlog/ Jira-ra tükrözve (Atlassian MCP, field-ownership)
│  └─ EXAMPLE.md
└─ pte-openspec-tdd-apply/
   └─ SKILL.md                     ← build stage: change taskjait teszt-first implementálja
```

Generált artefaktumok (nem ebben a mappában — az OpenSpec fán belül, a spec-ek mellett):

```
openspec/
├─ specs/            ← OpenSpec spec-ek (a lánc forrása)
├─ changes/          ← OpenSpec change-ek (delta spec + tasks)
└─ backlog/          ← a generált backlog (nem OpenSpec-CLI-kezelt)
   ├─ epics/         ← <epic-slug>.md
   └─ stories/       ← <epic-slug>/<story-slug>.md (a beágyazott BDD tesztekkel)
```

Külön `features/` fa **nincs**, és nincs gyökér-szintű `epics/`/`stories/` sem — minden az `openspec/backlog/**` alá kerül.

Minden **artefaktum-termelő** skillhez tartozik egy `EXAMPLE.md` a teljes kidolgozott átalakítással (be → ki, ID-kkal és map-pel); a `pte-openspec-tdd-apply` build stage kódot termel, nem artefaktumot, ezért nincs `EXAMPLE.md`-je. A `pte-openspec-bdd-tests` `BDD-RULES.md`-je a deklaratív Gherkin szabálykészletét és anti-pattern-jeit tartalmazza — a skill lépéslépés előtt betölti.
