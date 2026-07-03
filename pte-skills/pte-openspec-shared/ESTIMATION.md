# pte-openspec-* pipeline — the estimation model (PERT + Story Points reference base)

Single source of truth for the estimation math every story card and epic rollup obeys — a change here is a one-place edit. `pte-openspec-to-stories` authors each story's `## Estimation` against this file; `pte-openspec-to-epics` rolls the child estimates up. The ownership rule (which side owns the SP, how it syncs to Jira) lives in [`CONVENTIONS.md`](CONVENTIONS.md) alongside the pipeline's other ownership rules, not here.

Every story card carries an **`## Estimation`** section, and each epic carries a rolled-up one. The estimate is an **AI-authored reference base**: within the pipeline it is authoritative and **local-owned**. Managers layer their own internal/client multipliers on top **downstream** — that multiplier step is **out of pipeline scope**; these skills emit the raw base only, never a multiplied or client-facing number.

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

**σ-drivers — uncertainty sets the O↔P spread, not M** (score each **0–3**): követelmény-tisztaság (vague spec / open *Risks* / TBD) · tech-újdonság (first-time framework/service, spike needed) · domain-újdonság (unfamiliar business rules/process) · függőség-stabilitás (external system flaky / unversioned / uncontrolled). Let `u = Σ(σ-driver scores) / 12` (0..1), then `O = round(M · (1 − 0.4·u))` and `P = round(M · (1 + 2.0·u))`. So many unknowns → wide O/P → high σ → the split-warning can trip on uncertainty alone (the `σ/Eβ > 0.25` branch fires at `u > 0.75`, i.e. 10+ of the 12 σ-points); a clean, well-understood story has `u ≈ 0`, `O ≈ M ≈ P`, `σ ≈ 0`. (The `0.4`/`2.0` spread coefficients and `k` are tunable here alongside the weights — note the coefficients cap `σ/Eβ` at ~0.32, which is what calibrates the 0.25 threshold below.)

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

**Split-warning (advisory, flag-only — not a hard gate).** When `SP ≥ 13` **OR** `σ / Eβ > 0.25` (too big / too uncertain — the σ branch is calibrated to the spread coefficients above: their maximum is ~0.32 at `u = 1`, so 0.25 means *extreme* uncertainty, spike territory), the card still ships, but its `## Estimation` carries a visible `⚠` note (an extra line) that the story is a split candidate. This does **not** block emission — INVEST stays the qualitative gate (the estimate only surfaces an objective signal for the human to act on).

## Placement — pipeline-only, but always synced to Jira's structured fields

The `## Estimation` section lives **inside the `<!-- pipeline-only -->` fence** on every story card and epic (see *Pipeline-only blocks* in [`CONVENTIONS.md`](CONVENTIONS.md)), so it is **stripped from the Jira Description** — the estimate never clutters the pushed prose. It is **not** hidden from the pipeline: `pte-openspec-jira-sync` always reads it from the fenced block and pushes its numbers to Jira's **structured** fields (SP → *Story points*, `Becsült munkaóra`/`Eβ` → *Original estimate*) for both stories and epics. Author the section exactly as below; only its on-disk location (inside the fence) changed.

## Rendered `## Estimation` format — what the card and epic actually print

The story card and the epic render **different** shapes: the **story** prints the three-point (`3-Points`) estimate stacked, the **epic** prints only the summed rollup (a per-story `O`/`M`/`P` cannot be summed, so it collapses to the total `Becsült munkaóra`).

Story card — three stacked top-level bullets: the `3-Points` estimate **inline on one line** (`O`/`M`/`P`/`σ` side by side), then `Becsült munkaóra` (`Eβ` + ideal days), then `Story Point`:

```markdown
## Estimation
- **3-Points becslés (ideális óra):** O <O> / M <M> / P <P> | **Eβ <Eβ>**, σ <σ, 1 tizedes>
- **Becsült munkaóra:** <Eβ> ó (≈ <Eβ/6, 1 tizedes> ideális nap)
- **Story Point:** <SP>
```

Epic (rollup — summed over the child stories; **no** `3-Points` block, since `O`/`M`/`P` don't sum):

```markdown
## Estimation
- **Becsült munkaóra:** <Σ Eβ> ó (≈ <Σ Eβ/6, 1 tizedes> ideális nap)
- **Story Point:** <Σ SP>
```

Rules:
- **`3-Points becslés`** (story only) — the **canonical, final form** is one inline line, with exact separators:

  `- **3-Points becslés (ideális óra):** O <O> / M <M> / P <P> | **Eβ <Eβ>**, σ <σ>`

  In order: `O`, `M`, `P` joined by ` / `; then a ` | ` pipe before `Eβ`; **`Eβ` is bolded** (`**Eβ <Eβ>**`) because it is the value to rely on; then a `, ` comma before `σ`. All on **one** line side by side — never sub-bullets, and never without the `|` / bold-`Eβ` / comma. `σ` keeps its Hungarian decimal comma (e.g. `1,8`). `Eβ` also repeats as the `Becsült munkaóra` value (there with the ideal-day equivalent). The three top-level bullets (`3-Points becslés`, `Becsült munkaóra`, `Story Point`) stay stacked.
- **`Becsült munkaóra`** = the `Eβ` ideal engineer-hours (story) or `Σ Eβ` (epic), with the ideal-day equivalent in parentheses (anchor `6 ó = 1 ideális nap`; round the day figure to 1 decimal with a Hungarian comma, and drop a trailing `,0` so `1,0` → `1`).
- **`Story Point`** = the derived SP as a **bare integer** (e.g. `3`) — no ` Points` suffix, no `Σ` prefix, and no story-count suffix.
- The `⚠` split-warning (when `SP ≥ 13` or `σ/Eβ > 0.25`) is still appended as an extra line when it trips.

**Epic rollup.** An epic's `## Estimation` is the **sum of its child stories'** `Eβ` hours and SP, filled by `pte-openspec-to-epics` **only once every child story is estimated**. If any child card lacks an estimate, mark the rollup incomplete (`⚠ nem minden story esztimált`) rather than guessing a total.
