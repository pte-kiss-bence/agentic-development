# Context map

This repo models its domains as bounded contexts (see the `domain-modeling` skill): each context owns a `CONTEXT.md` glossary; decisions live in ADRs. Today there is **one** modelled context; application code built from this template adds its own contexts as new rows.

| Context | Scope | Glossary | Decisions |
|---------|-------|----------|-----------|
| **pte-openspec pipeline** | the `pte-skills/` skill family: OpenSpec specs → epics → story cards → BDD tests → Jira, plus the test-first build stage | [`pte-skills/CONTEXT.md`](pte-skills/CONTEXT.md) | [`docs/adr/`](docs/adr/) 0001–0005 |

- `docs/adr/` currently holds the pipeline's decisions and any future system-wide ones; split out per-context `docs/adr/` directories when a second context lands.
- A future app capability gets its own `<dir>/CONTEXT.md` and a row here — do not grow a second glossary for an existing context.
