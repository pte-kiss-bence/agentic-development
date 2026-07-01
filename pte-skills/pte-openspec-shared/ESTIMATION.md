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

**Epic rollup.** An epic's `## Estimation` is the **sum of its child stories'** `Eβ` hours and SP, filled by `pte-openspec-to-epics` **only once every child story is estimated**. If any child card lacks an estimate, mark the rollup incomplete (`⚠ nem minden story esztimált`) rather than guessing a total.
