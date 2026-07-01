# pte-openspec-* pipeline — shared conventions

Single source of truth for the rules every `pte-openspec-*` skill obeys, so a change to a shared rule is a one-place edit. The pipeline: `openspec` explore/propose → **pte-openspec-to-epics** → **pte-openspec-to-stories** → **pte-openspec-bdd-tests**. Each skill links here instead of restating these rules.

## Output language

Every generated **artifact's content is written in Hungarian**, matching `openspec/config.yaml`'s artifact convention. Keep verbatim — do **not** translate: code, identifiers, API names, CLI commands, file paths, `### Requirement` names, `#### Scenario` titles, `EPIC-…`/`STORY-…` trace IDs/slugs, and commit-type keywords (`feat`/`fix`/...). Section headings prescribed by a skill's anatomy are already Hungarian; use them as-is. The skills themselves (`SKILL.md` prose) stay in English — only the files they emit are Hungarian.

## Trace-ID grammar and ownership

- `EPIC-<epic-slug>` — `epic-slug` = kebab-case of the epic title (capability / requirement-group theme).
- `STORY-<epic-slug>-<requirement-slug>` — `requirement-slug` = kebab-case of the story's primary source `### Requirement` name. Anchored on the requirement, **not** a running index, so the ID is stable when stories are reordered.
- When several stories split the same primary requirement, append a stable `-<aspect-slug>` derived from the story's value/journey (e.g. `STORY-checkout-fizetes-utalas`, `STORY-checkout-fizetes-kartya`) — never a positional number; the aspect-slug names the slice, so it survives reordering too.
- IDs and slugs stay verbatim even when the surrounding content is Hungarian. Each `STORY-…` is unique within its epic.

**Ownership:** `pte-openspec-to-epics` **MINTS** these IDs. `pte-openspec-to-stories` and `pte-openspec-bdd-tests` **CONSUME** them — reference verbatim, never re-mint.

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

## Writing output files

Diff before overwriting — never clobber hand-edited content. One artifact per file, under the skill's configured output directory.
