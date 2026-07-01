# pte-openspec-* pipeline — shared conventions

Single source of truth for the rules every `pte-openspec-*` skill obeys, so a change to a shared rule is a one-place edit. The **planning chain is linear** (the build stage `pte-openspec-tdd-execute` is a separate phase off the same spec — see the router) — `openspec` explore/propose → **pte-openspec-to-epics** (→ `openspec/backlog/epics/`) → **pte-openspec-to-stories** (→ `openspec/backlog/stories/`) → **pte-openspec-bdd-tests** (embeds Gherkin into the story cards) → optional **pte-openspec-jira-sync** (mirrors the backlog to Jira). One **optional, read-only view** hangs off the backlog without extending the chain: **pte-openspec-dependency-graph** renders the `## Dependency Edges` as a Mermaid graph. Two artifact kinds only: the **epic** carries its stories in its description; the **story card** carries its own BDD test(s) in its description. There is no separate `features/` tree. Each skill links here instead of restating these rules. Term definitions (the pipeline's ubiquitous language — *epic*, *trace ID*, *MINTS/CONSUMES*, *mint/reconcile/epic-less*, *field-ownership*, *reference base*, …) live in [`docs/glossary.md`](../../docs/glossary.md).

The chain is **right-sized to the change**, not always run whole: a full feature runs `to-epics` in **mint** mode; a change request that lands in an **existing** epic runs `to-epics` in **reconcile** mode (fold the new story into that epic); a small standalone change that warrants a story but no epic **skips `to-epics`** and enters at `to-stories`'s **epic-less mode**. All three converge on the same `to-stories` → `bdd-tests` → (`tdd-apply`) → `jira-sync` tail.

## Output language

Every generated **artifact's content is written in Hungarian**, matching `openspec/config.yaml`'s artifact convention. Keep verbatim — do **not** translate: code, identifiers, API names, CLI commands, file paths, `### Requirement` names, `#### Scenario` titles, `EPIC-…`/`STORY-…` trace IDs/slugs, and commit-type keywords (`feat`/`fix`/...). The skills themselves (`SKILL.md` prose) stay in English — only the files they emit are Hungarian.

**Section headings are English; body is Hungarian.** The epic and story-card anatomies use **English `##` headings** (epic: `Epic`, `Persona`, `E2E Scenario`, `Problem / Solution`, `Cross-cutting Concerns`, `MVP and Out of Scope`, `Success metrics`, `Risks and Dependencies`, `High Level Acceptance Criteria`; story: `User Story`, `Context`, `BDD Test`, `Risks and Dependencies`) — use them verbatim. **All `##` headings are English — no exceptions.** This now includes the pipeline-only headings (`User Stories`, `Source Spec Reference`, `Source Reference`) and the `pte-openspec-jira-sync`-owned `Jira Sync` block; only the prose *under* each heading stays Hungarian. The epic's main narrative section is **`## Epic`** and the story's is **`## User Story`** (never `## Description`). Inside the epic's `## Problem / Solution`, the two labels are spelled out **and bold** — `**Probléma:**` and `**Megoldás:**` — on separate stacked paragraphs, never the `P:`/`S:` shorthand. Likewise the epic's `## MVP and Out of Scope` labels its two lists with bold `**MVP:**` and `**Out of Scope:**`.

## Pipeline-only (hidden-from-Jira) blocks

Some content the pipeline needs — the epic's *Source Spec Reference* map and *User Stories* list, an epic-less card's self-carried *Source Reference* row, every artifact's **`## Estimation`** section, and every artifact's **`## Dependency Edges`** section — must **not** reach Jira **as Description text**. Wrap it in a single fenced region so `pte-openspec-jira-sync` can strip it by one boundary:

```markdown
<!-- pipeline-only:start -->
…pipeline-internal sections…
<!-- pipeline-only:end -->
```

The fence lives at the **end of the file** (after the last Jira-visible section). `pte-openspec-jira-sync` removes the whole region — markers inclusive — from the body it pushes as the Jira Description; the block stays on disk for the downstream skills. Everything outside the fence is Jira-visible.

**`## Estimation` is pipeline-only but still drives Jira's structured fields.** The estimate is deliberately kept **out of the Jira Description** (it lives inside the fence), yet `pte-openspec-jira-sync` **always reads it from the fenced block** and pushes its numbers to Jira's **structured** fields — the SP to *Story points* and the `Becsült munkaóra` (`Eβ`) to *Original estimate* — for **both** epics (rollup sums) and stories. So the estimate never clutters the Description prose, but the board's structured fields are always populated. This is the one pipeline-only section whose content maps to a Jira field; the *Source Spec Reference* / *User Stories* / *Source Reference* parts stay purely internal.

## Trace-ID grammar and ownership

- `EPIC-<epic-slug>` — `epic-slug` = kebab-case of the epic title (capability / requirement-group theme).
- `STORY-<epic-slug>-<requirement-slug>` — `requirement-slug` = kebab-case of the story's primary source `### Requirement` name. Anchored on the requirement, **not** a running index, so the ID is stable when stories are reordered.
- When several stories split the same primary requirement, append a stable `-<aspect-slug>` derived from the story's value/journey (e.g. `STORY-checkout-fizetes-utalas`, `STORY-checkout-fizetes-kartya`) — never a positional number; the aspect-slug names the slice, so it survives reordering too.
- **Epic-less story** — `STORY-<capability-slug>-<requirement-slug>`, namespaced on the **capability** (from `openspec/specs/<capability>/`) instead of an epic. Minted by `pte-openspec-to-stories`'s epic-less mode for a small standalone change that warrants a story but no epic. It is a **first-class, stable** ID (not provisional): the capability slug is as durable as an epic slug. Should the capability later grow an epic, the story keeps its ID and the epic adopts it by listing the row.
- IDs and slugs stay verbatim even when the surrounding content is Hungarian. Each `STORY-…` is unique within its epic (or, for an epic-less story, within its capability).

**Ownership:** `pte-openspec-to-epics` **MINTS** `EPIC-…` and their child `STORY-…`. In **reconcile** mode it reuses an existing epic's `EPIC-…` verbatim and mints only the new `STORY-…` a delta adds — it never re-mints an epic ID. `pte-openspec-to-stories` **CONSUMES** these IDs, except in **epic-less mode**, where it mints the `STORY-<capability-slug>-…` above. `pte-openspec-bdd-tests` only **CONSUMES** — reference verbatim, never re-mint.

## The *Source Spec Reference* map — cross-skill contract

`pte-openspec-to-epics` emits, per epic, a traceability table — the contract downstream skills read instead of re-deriving the requirement→story split:

| Story ID | Epic ID | Source `### Requirement` | Covered `#### Scenario`s |
|----------|---------|--------------------------|----------------------------|

Downstream skills match rows by **verbatim** `### Requirement` / `#### Scenario` title. They take only the IDs and the split from the map; the behavioural text (`WHEN`/`THEN`) is always sourced from the spec itself, never from epic prose.

## The *Dependency Edges* block — the edge contract

`ownership` governs edges the same way it governs fields: every artifact **owns its own outbound edges**, declared machine-readably in a `## Dependency Edges` section so two skills can read one source instead of parsing prose. `pte-openspec-dependency-graph` renders it as a Mermaid graph; `pte-openspec-jira-sync` mirrors it as real Jira issue links. The Jira-visible `## Risks and Dependencies` section stays **human narrative** — it is **never parsed** for edges, so the two never fight over the same text.

The section lives **inside the `<!-- pipeline-only -->` fence** on every epic and story card (stripped from the Jira Description, like `## Estimation`). Each edge is one bullet: an **ASCII keyword tag** (language-independent, so the parser never depends on Hungarian prose) then a single **backticked target trace ID**, written **from the card's own perspective**. Four tags:

```markdown
<!-- pipeline-only:start -->
## Dependency Edges
- depends-on: `EPIC-excel-ingest`
- blocks: `STORY-riport-export`
- relates-to: `EPIC-idoszak-lezaras`
- external: Entra ID tenant + szerepkör-hozzárendelés
<!-- pipeline-only:end -->
```

- **`depends-on: <TRACE-ID>`** — this card waits for the target (the target blocks this card).
- **`blocks: <TRACE-ID>`** — this card blocks the target.
- **`relates-to: <TRACE-ID>`** — a non-blocking coupling.
- **`external: <free text>`** — a dependency with no trace ID (an outside system, a team, a tenant). A **graph-only node**, never a Jira link; the only tag whose payload is free prose, not a backticked ID.

Rules the authoring skills (`pte-openspec-to-epics`, `pte-openspec-to-stories`) and the readers (`pte-openspec-dependency-graph`, `pte-openspec-jira-sync`) all obey:

- **Both directions are allowed** — `depends-on` and `blocks` are inverses (X `depends-on` Y states the same edge as Y `blocks` X). A reader **dedupes an inverse pair into one edge**; it never emits two.
- **Bare bullets are not edges.** A `## Risks and Dependencies` bullet with no tag (a plain `Kockázat:` line) is narrative, not an edge — readers skip it. Only the four tags above, in the `## Dependency Edges` section, count.
- **Dangling target = warn and skip.** A `depends-on`/`blocks`/`relates-to` whose trace ID is not present anywhere in the backlog (typo, or a deleted card) is **warned and skipped** — never a crash, never a fabricated node.
- **The section is the single source of truth for edges.** The authoring skills derive it best-effort (from the spec's requirement cross-references and the `## Risks and Dependencies` narrative they already write); a human refines it. Nothing re-derives edges from prose downstream.

## Estimation — the reference base (PERT + Story Points)

Every story card carries an **`## Estimation`** section, and each epic a rolled-up one — an **AI-authored reference base** the managers' downstream internal/client multipliers ride on (that multiplier step is out of pipeline scope; these skills emit the raw base only). The full model — PERT three-point in ideal engineer-hours, the weighted complexity rubric (M-drivers/modifiers, σ-drivers), the tuning constants (`k`, spread coefficients), the Eβ→SP table, the split-warning threshold, and the epic rollup rule — is the single source of truth in [`ESTIMATION.md`](ESTIMATION.md); `pte-openspec-to-stories` and `pte-openspec-to-epics` author against it. Only the **ownership** of the resulting numbers lives here, next to the pipeline's other ownership rules:

**Ownership.** The SP is **local-owned** and pushed to Jira's structured *Story points* field (Local → Jira) — this **supersedes** any earlier "estimate is Jira-owned" framing. A **story** card's visible `## Estimation` renders the inline **`3-Points becslés`** (`O`/`M`/`P`/`Eβ`/`σ` on one line), then **`Becsült munkaóra`** (`Eβ` + ideal days), then **`Story Point`** (`<SP>`, a bare integer); an **epic** rollup renders only `Becsült munkaóra` + `Story Point` (no `3-Points` block — `O`/`M`/`P` don't sum). All these lines reach the Jira Description, but only the SP maps to the structured field. The exact rendered format (story + epic-rollup) lives in [`ESTIMATION.md`](ESTIMATION.md).

## INVEST — the story gate

Every story card **must** satisfy INVEST — **I**ndependent, **N**egotiable, **V**aluable, **E**stimable, **S**mall, **T**estable. It is a **rule, not a printed section**: the lean card no longer carries an *INVEST-ellenőrzés* block, but conformance is a hard, silent gate checked while authoring. A slice that cannot be made INVEST-conform is a signal to **re-cut the split**, never to ship a non-conforming card:

- `pte-openspec-to-epics` owns the split, so it cuts every story INVEST-conform up front (step 4).
- `pte-openspec-to-stories` re-checks all six letters per card. In epic-driven mode a story that only becomes conform by re-splitting is kicked back to `pte-openspec-to-epics` (the epic owns the split); in epic-less mode the reshape happens in place.

INVEST is the **qualitative** gate; the estimation *split-warning* (`ESTIMATION.md`) is only an objective signal that feeds the same judgement, not a second gate.

## Source-spec resolution

Resolve which spec to read:
- A change's delta: `openspec list --json` → pick the change → `openspec/changes/<id>/specs/**/spec.md`.
- A main spec: `openspec/specs/<capability>/spec.md`.
- Pass `--store <id>` on `openspec` commands when the user names a store.
- If the input is vague, list the options with **AskUserQuestion**.

## Output location

The generated backlog lives **under the OpenSpec tree**, co-located with the specs it traces from — the canonical default (configurable) is `openspec/backlog/`, laid out as below.

Both artifact kinds are filed under their own trace ID, so the filename **is** the ID and a single `grep`/glob finds an `EPIC-…`/`STORY-…` by name across the tree:

- **Epics** → `openspec/backlog/epics/EPIC-<epic-slug>.md` — the filename carries the full `EPIC-<epic-slug>` trace ID. The `<epic-slug>` used to pair the story directory is the filename **minus** its `EPIC-` prefix.
- **Story cards** → `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md` — the filename carries the full `STORY-…` trace ID (the same one in the card's *Cím* and *Source Reference* row), giving searchability parity with the `EPIC-…` files. The parent `<epic-slug>` directory repeats inside the filename by construction, since the `STORY-…` ID is epic-namespaced.
- **Epic-less story cards** (no parent epic) → `openspec/backlog/stories/<capability-slug>/STORY-<capability-slug>-<requirement-slug>.md` (the `STORY-…` ID is capability-namespaced instead of epic-namespaced; the filename is still the full trace ID).

`openspec/backlog/` is **not** OpenSpec-CLI-managed (unlike `openspec/specs/` and `openspec/changes/`) — `openspec update`/`archive` never touch it — but it is deliberately kept inside `openspec/` so the backlog and its source specs travel together. Every skill's default output path resolves under here; when a skill says "default `openspec/backlog/epics/`, configurable", this is the definition. Nothing is written outside `openspec/backlog/**` — there is no separate root-level `epics/`/`stories/` or `features/` tree.

## Writing output files

Diff before overwriting — never clobber hand-edited content. `pte-openspec-to-epics` and `pte-openspec-to-stories` write one artifact per file under `openspec/backlog/` (see *Output location*). `pte-openspec-bdd-tests` writes nothing new — it edits the existing story cards in place, filling only their `BDD Test` section and leaving every other section untouched. `pte-openspec-jira-sync` likewise edits epics and cards in place, writing only their `## Jira Sync` block (Jira-owned fields pulled back from the board), and additionally mirrors the artifacts to Jira.
