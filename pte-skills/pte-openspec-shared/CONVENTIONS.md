# pte-openspec-* pipeline — shared conventions

Single source of truth for the rules every `pte-openspec-*` skill obeys, so a change to a shared rule is a one-place edit. The pipeline is **linear** — `openspec` explore/propose → **pte-openspec-to-epics** (→ `openspec/backlog/epics/`) → **pte-openspec-to-stories** (→ `openspec/backlog/stories/`) → **pte-openspec-bdd-tests** (embeds Gherkin into the story cards) → optional **pte-openspec-jira-sync** (mirrors the backlog to Jira). Two artifact kinds only: the **epic** carries its stories in its description; the **story card** carries its own BDD test(s) in its description. There is no separate `features/` tree. Each skill links here instead of restating these rules.

The chain is **right-sized to the change**, not always run whole: a full feature runs `to-epics` in **mint** mode; a change request that lands in an **existing** epic runs `to-epics` in **reconcile** mode (fold the new story into that epic); a small standalone change that warrants a story but no epic **skips `to-epics`** and enters at `to-stories`'s **epic-less mode**. All three converge on the same `to-stories` → `bdd-tests` → (`tdd-apply`) → `jira-sync` tail.

## Output language

Every generated **artifact's content is written in Hungarian**, matching `openspec/config.yaml`'s artifact convention. Keep verbatim — do **not** translate: code, identifiers, API names, CLI commands, file paths, `### Requirement` names, `#### Scenario` titles, `EPIC-…`/`STORY-…` trace IDs/slugs, and commit-type keywords (`feat`/`fix`/...). Section headings prescribed by a skill's anatomy are already Hungarian; use them as-is. The skills themselves (`SKILL.md` prose) stay in English — only the files they emit are Hungarian.

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

## Source-spec resolution

Resolve which spec to read:
- A change's delta: `openspec list --json` → pick the change → `openspec/changes/<id>/specs/**/spec.md`.
- A main spec: `openspec/specs/<capability>/spec.md`.
- Pass `--store <id>` on `openspec` commands when the user names a store.
- If the input is vague, list the options with **AskUserQuestion**.

## Output location

The generated backlog lives **under the OpenSpec tree**, co-located with the specs it traces from — the canonical default (configurable) is:

- **Epics** → `openspec/backlog/epics/EPIC-<epic-slug>.md` — the filename carries the `EPIC-` prefix for searchability parity with the `STORY-…` cards. The `<epic-slug>` used to pair the story directory is the filename **minus** its `EPIC-` prefix.
- **Story cards** → `openspec/backlog/stories/<epic-slug>/<story-slug>.md`
- **Epic-less story cards** (no parent epic) → `openspec/backlog/stories/<capability-slug>/<story-slug>.md` (namespaced on the capability instead of an epic).

`openspec/backlog/` is **not** OpenSpec-CLI-managed (unlike `openspec/specs/` and `openspec/changes/`) — `openspec update`/`archive` never touch it — but it is deliberately kept inside `openspec/` so the backlog and its source specs travel together. Every skill's default output path resolves under here; when a skill says "default `openspec/backlog/epics/`, configurable", this is the definition. Nothing is written outside `openspec/backlog/**` — there is no separate root-level `epics/`/`stories/` or `features/` tree.

## Writing output files

Diff before overwriting — never clobber hand-edited content. `pte-openspec-to-epics` and `pte-openspec-to-stories` write one artifact per file under `openspec/backlog/` (see *Output location*). `pte-openspec-bdd-tests` writes nothing new — it edits the existing story cards in place, filling only their `BDD teszt` section and leaving every other section untouched. `pte-openspec-jira-sync` likewise edits epics and cards in place, writing only their `## Jira szinkron` block (Jira-owned fields pulled back from the board), and additionally mirrors the artifacts to Jira.
