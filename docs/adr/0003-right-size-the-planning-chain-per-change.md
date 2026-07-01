# pte-openspec: the planning chain is right-sized per change, not always run whole

A change request is not always a feature. It may be a full new capability, a modification of existing behaviour, a single small standalone story, or a bug. Running the whole planning chain (`pte-openspec-to-epics` → `-to-stories` → `-bdd-tests`) on every one of these forces epics and story cards onto work that has nothing to decompose — pure ceremony. So the chain is **entered at the depth the change warrants**, and the post-propose offer (after `openspec-propose` / `/opsx:propose`, pinned in `CLAUDE.md`) presents that choice rather than a yes/no.

Four right-sized paths, converging on one tail (`to-stories → bdd-tests → tdd-apply → jira-sync`):

- **Full feature** → `pte-openspec-to-epics` in **mint** mode (greenfield: a new capability with no epic yet), then the whole chain.
- **Change into an existing epic** (the common case, since a change request modifies existing behaviour) → `pte-openspec-to-epics` in **reconcile** mode: resolve the epic whose *Forrás-spec hivatkozás* rows trace to the delta's capability, reuse its `EPIC-…` verbatim, mint a new `STORY-…` for each added requirement, extend an existing story's coverage for a modified scenario, and leave everything untouched byte-for-byte (diff-don't-clobber).
- **Small standalone change** (a story, no epic) → skip `to-epics`; `pte-openspec-to-stories` runs in **epic-less** mode, authoring the card(s) straight from the delta and minting its own capability-namespaced `STORY-<capability-slug>-…` (stable, not provisional), with the parent `EPIC-…` line left empty.
- **Tweak / bug** → skip planning entirely, straight to `pte-openspec-tdd-apply` (test-first; for a bug, run `diagnosing-bugs` first when the root cause isn't obvious).

Two decisions worth recording explicitly:

**Reconcile lives in `pte-openspec-to-epics`, not a separate skill.** The delta→epic knowledge (capability grouping, the traceability-map contract, the ID grammar) already lives there; a second skill would duplicate it and risk drift. One skill, two modes, mode resolved by whether an epic already covers the delta's capability.

**Epic-less stories are first-class, not a degraded fallback.** The earlier `to-stories` no-epic path minted "provisional" IDs "until an epic reconciles them"; for a genuinely standalone small change no epic ever comes, so provisional-forever was the wrong framing. The capability slug is as durable an anchor as an epic slug, so the ID is stable. To keep the Jira mirror consistent, `pte-openspec-jira-sync` parents epic-less stories under a single reserved **standalone collector epic** (`trace:EPIC-standalone`) so they are never orphaned on the board.

The cost is that the entry point is now a judgement call (which mode?) rather than a fixed step. That judgement is kept mechanical — *does an epic already cover this capability?* yes → reconcile, no-but-needed → mint, no-and-not-needed → epic-less — and the router (`pte-openspec/SKILL.md`) plus the post-propose offer (pinned in `CLAUDE.md`, which survives `openspec update`) encode it so the choice is consistent across the team.
