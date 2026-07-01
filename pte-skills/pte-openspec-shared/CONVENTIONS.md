# pte-openspec-* pipeline — shared conventions

Single source of truth for the rules every `pte-openspec-*` skill obeys, so a change to a shared rule is a one-place edit. The pipeline is **linear** — `openspec` explore/propose → **pte-openspec-to-epics** (→ `openspec/backlog/epics/`) → **pte-openspec-to-stories** (→ `openspec/backlog/stories/`) → **pte-openspec-bdd-tests** (embeds Gherkin into the story cards) → optional **pte-openspec-jira-sync** (mirrors the backlog to Jira). Two artifact kinds only: the **epic** carries its stories in its description; the **story card** carries its own BDD test(s) in its description. There is no separate `features/` tree. Each skill links here instead of restating these rules.

The chain is **right-sized to the change**, not always run whole: a full feature runs `to-epics` in **mint** mode; a change request that lands in an **existing** epic runs `to-epics` in **reconcile** mode (fold the new story into that epic); a small standalone change that warrants a story but no epic **skips `to-epics`** and enters at `to-stories`'s **epic-less mode**. All three converge on the same `to-stories` → `bdd-tests` → (`tdd-apply`) → `jira-sync` tail.

## Output language

Every generated **artifact's content is written in Hungarian**, matching `openspec/config.yaml`'s artifact convention. Keep verbatim — do **not** translate: code, identifiers, API names, CLI commands, file paths, `### Requirement` names, `#### Scenario` titles, `EPIC-…`/`STORY-…` trace IDs/slugs, and commit-type keywords (`feat`/`fix`/...). The skills themselves (`SKILL.md` prose) stay in English — only the files they emit are Hungarian.

**Section headings are English; body is Hungarian.** The epic and story-card anatomies use **English `##` headings** (epic: `Description`, `Persona`, `E2E Scenario`, `Problem / Solution`, `Cross-cutting Concerns`, `MVP and Out of Scope`, `Success metrics`, `Risks and Dependencies`, `High Level Acceptance Criteria`; story: `Description`, `Context`, `BDD Test`, `Risks and Dependencies`) — use them verbatim. The only **Hungarian** headings left are the pipeline-internal ones that never reach Jira: everything inside a **pipeline-only** block (`User story-k`, `Forrás-spec hivatkozás`, `Forrás-hivatkozás`) and the `pte-openspec-jira-sync`-owned `Jira szinkron` block.

## Pipeline-only (hidden-from-Jira) blocks

Some content the pipeline needs — the epic's *Forrás-spec hivatkozás* map and *User story-k* list, an epic-less card's self-carried *Forrás-hivatkozás* row — must **not** reach Jira. Wrap it in a single fenced region so `pte-openspec-jira-sync` can strip it by one boundary:

```markdown
<!-- pipeline-only:start -->
…pipeline-internal sections…
<!-- pipeline-only:end -->
```

The fence lives at the **end of the file** (after the last Jira-visible section). `pte-openspec-jira-sync` removes the whole region — markers inclusive — from the body it pushes as the Jira Description; the block stays on disk for the downstream skills. Everything outside the fence is Jira-visible.

## Trace-ID grammar and ownership

- `EPIC-<epic-slug>` — `epic-slug` = kebab-case of the epic title (capability / requirement-group theme).
- `STORY-<epic-slug>-<requirement-slug>` — `requirement-slug` = kebab-case of the story's primary source `### Requirement` name. Anchored on the requirement, **not** a running index, so the ID is stable when stories are reordered.
- When several stories split the same primary requirement, append a stable `-<aspect-slug>` derived from the story's value/journey (e.g. `STORY-checkout-fizetes-utalas`, `STORY-checkout-fizetes-kartya`) — never a positional number; the aspect-slug names the slice, so it survives reordering too.
- **Epic-less story** — `STORY-<capability-slug>-<requirement-slug>`, namespaced on the **capability** (from `openspec/specs/<capability>/`) instead of an epic. Minted by `pte-openspec-to-stories`'s epic-less mode for a small standalone change that warrants a story but no epic. It is a **first-class, stable** ID (not provisional): the capability slug is as durable as an epic slug. Should the capability later grow an epic, the story keeps its ID and the epic adopts it by listing the row.
- IDs and slugs stay verbatim even when the surrounding content is Hungarian. Each `STORY-…` is unique within its epic (or, for an epic-less story, within its capability).

**Ownership:** `pte-openspec-to-epics` **MINTS** `EPIC-…` and their child `STORY-…`. In **reconcile** mode it reuses an existing epic's `EPIC-…` verbatim and mints only the new `STORY-…` a delta adds — it never re-mints an epic ID. `pte-openspec-to-stories` **CONSUMES** these IDs, except in **epic-less mode**, where it mints the `STORY-<capability-slug>-…` above. `pte-openspec-bdd-tests` only **CONSUMES** — reference verbatim, never re-mint.

## The *Forrás-spec hivatkozás* map — cross-skill contract

`pte-openspec-to-epics` emits, per epic, a traceability table — the contract downstream skills read instead of re-deriving the requirement→story split:

| Story ID | Epic ID | Forrás `### Requirement` | Lefedett `#### Scenario`-k |
|----------|---------|--------------------------|----------------------------|

Downstream skills match rows by **verbatim** `### Requirement` / `#### Scenario` title. They take only the IDs and the split from the map; the behavioural text (`WHEN`/`THEN`) is always sourced from the spec itself, never from epic prose.

## Estimation — the reference base (PERT + Story Points)

Every story card carries an **`## Estimation`** section, and each epic carries a rolled-up one. The estimate is an **AI-authored reference base**: within the pipeline it is authoritative and **local-owned**. Managers layer their own internal/client multipliers on top **downstream** — that multiplier step is **out of pipeline scope**; these skills emit the raw base only, never a multiplied or client-facing number. Single source of truth for the math (a change here is a one-place edit; the skills reference this):

**PERT three-point.** Estimate each story with three points in **ideal engineer-hours** (focused dev, no meetings/buffer — that overhead is exactly what the downstream multipliers add):

- `O` = optimista, `M` = valószínű (most likely), `P` = pesszimista (whole hours).
- **Eβ** (várható) `= (O + 4·M + P) / 6` — the reference base number the multipliers ride on. Round to the nearest hour.
- **σ** (bizonytalanság) `= (P − O) / 6`.

**Anchor: 6 ideal engineer-hours = 1 ideal day.**

**Where O/M/P come from — the weighted complexity rubric.** Do **not** guess the hours cold. Score the story against a fixed set of factors read from the card and its source spec, so every run follows the same *process*. This is a **weighted-factor (parametric) model**, not reference-class analogy — no historical anchor set is needed; the weights and the single scale-constant `k` are the tuning knobs, and they live **here** so a recalibration is a one-place edit.

**M-drivers** — score each **0–3** (nincs / kicsi / közepes / nagy), multiply by its weight:

| Factor | Signal (card / spec) | Weight |
|--------|----------------------|--------|
| Viselkedés-volumen | `#### Scenario` count, `WHEN`/`THEN` + error branches | ×3 |
| Üzleti-logika komplexitás | rules, calculation, state machine, conditionals (≠ scenario count) | ×3 |
| Integráció / csatolás | external systems + internal sibling-story deps + async/eventing | ×3 |
| Interfész-felület | UI (screens/forms/states) **+** API/contract (endpoints, DTOs, versioning) | ×2 |
| Adat / perzisztencia | new entity, migration, schema, data volume | ×2 |
| Teszt / QA-erőfeszítés | test build (scenarios → later Playwright) + review/sign-off rounds | ×2 |
| Deploy / infra / DevOps | new env, CI/CD step, feature-flag, rollout mechanics | ×2 |

**M-modifiers** — added to `S` **only when the story hits them** (score **0–2**, weight ×1, so they don't tax a typical small story): biztonság/jogosultság · compliance/audit-napló (incl. adatvédelem/GDPR) · teljesítmény/skála · i18n · migráció/backfill/fázisolt rollout · observability · dokumentáció.

Then `S = Σ(M-driver score × weight) + Σ(modifier scores)`, and **`M = round(S × k)`**, with **`k = 0.6` ideal engineer-hours / point** (default). Calibration intent: a tiny story lands ~1–2 SP, a typical mid story ~5–8 SP, and a story scoring near the ceiling on every dimension reaches ~13 SP — i.e. the rubric itself pushes an oversized story into the `⚠` split zone.

**σ-drivers — uncertainty sets the O↔P spread, not M** (score each **0–3**): követelmény-tisztaság (vague spec / open *Risks* / TBD) · tech-újdonság (first-time framework/service, spike needed) · domain-újdonság (unfamiliar business rules/process) · függőség-stabilitás (external system flaky / unversioned / uncontrolled). Let `u = Σ(σ-driver scores) / 12` (0..1), then `O = round(M · (1 − 0.4·u))` and `P = round(M · (1 + 2.0·u))`. So many unknowns → wide O/P → high σ → the split-warning can trip on uncertainty alone; a clean, well-understood story has `u ≈ 0`, `O ≈ M ≈ P`, `σ ≈ 0`. (The `0.4`/`2.0` spread coefficients and `k` are tunable here alongside the weights.)

**Story Points** (modified Fibonacci) are **derived deterministically from Eβ** by this fixed table — never guessed independently, so the two views can't contradict:

| Eβ (ideal engineer-hours) | SP | ~ |
|---------------------------|----|----|
| ≤ 2 | 1 | ~⅓ nap |
| ≤ 4 | 2 | ~⅔ nap |
| ≤ 6 | 3 | ~1 nap |
| ≤ 12 | 5 | ~2 nap |
| ≤ 30 | 8 | ~1 hét |
| ≤ 60 | 13 | ~2 hét |
| > 60 | 20 | — |

**Split-warning (advisory, flag-only — not a hard gate).** When `SP ≥ 13` **OR** `σ / Eβ > 0.5` (too big / too uncertain), the card still ships, but its `## Estimation` carries a visible `⚠` note that the story is a split candidate. This does **not** block emission — INVEST stays the qualitative gate (the estimate only surfaces an objective signal for the human to act on).

**Ownership.** The SP is **local-owned** and pushed to Jira's structured *Story points* field (Local → Jira) — this **supersedes** any earlier "estimate is Jira-owned" framing. The PERT base (`O`/`M`/`P`, `Eβ`, `σ`) lives in the visible `## Estimation` section (so it also reaches the Jira Description), but only the SP maps to the structured field.

**Epic rollup.** An epic's `## Estimation` is the **sum of its child stories'** `Eβ` hours and SP, filled by `pte-openspec-to-epics` **only once every child story is estimated**. If any child card lacks an estimate, mark the rollup incomplete (`⚠ nem minden story esztimált`) rather than guessing a total.

## Source-spec resolution

Resolve which spec to read:
- A change's delta: `openspec list --json` → pick the change → `openspec/changes/<id>/specs/**/spec.md`.
- A main spec: `openspec/specs/<capability>/spec.md`.
- Pass `--store <id>` on `openspec` commands when the user names a store.
- If the input is vague, list the options with **AskUserQuestion**.

## Output location

The generated backlog lives **under the OpenSpec tree**, co-located with the specs it traces from — the canonical default (configurable) is:

Both artifact kinds are filed under their own trace ID, so the filename **is** the ID and a single `grep`/glob finds an `EPIC-…`/`STORY-…` by name across the tree:

- **Epics** → `openspec/backlog/epics/EPIC-<epic-slug>.md` — the filename carries the full `EPIC-<epic-slug>` trace ID. The `<epic-slug>` used to pair the story directory is the filename **minus** its `EPIC-` prefix.
- **Story cards** → `openspec/backlog/stories/<epic-slug>/STORY-<epic-slug>-<requirement-slug>.md` — the filename carries the full `STORY-…` trace ID (the same one in the card's *Cím* and *Forrás-hivatkozás* row), giving searchability parity with the `EPIC-…` files. The parent `<epic-slug>` directory repeats inside the filename by construction, since the `STORY-…` ID is epic-namespaced.
- **Epic-less story cards** (no parent epic) → `openspec/backlog/stories/<capability-slug>/STORY-<capability-slug>-<requirement-slug>.md` (the `STORY-…` ID is capability-namespaced instead of epic-namespaced; the filename is still the full trace ID).

`openspec/backlog/` is **not** OpenSpec-CLI-managed (unlike `openspec/specs/` and `openspec/changes/`) — `openspec update`/`archive` never touch it — but it is deliberately kept inside `openspec/` so the backlog and its source specs travel together. Every skill's default output path resolves under here; when a skill says "default `openspec/backlog/epics/`, configurable", this is the definition. Nothing is written outside `openspec/backlog/**` — there is no separate root-level `epics/`/`stories/` or `features/` tree.

## Writing output files

Diff before overwriting — never clobber hand-edited content. `pte-openspec-to-epics` and `pte-openspec-to-stories` write one artifact per file under `openspec/backlog/` (see *Output location*). `pte-openspec-bdd-tests` writes nothing new — it edits the existing story cards in place, filling only their `BDD Test` section and leaving every other section untouched. `pte-openspec-jira-sync` likewise edits epics and cards in place, writing only their `## Jira szinkron` block (Jira-owned fields pulled back from the board), and additionally mirrors the artifacts to Jira.
