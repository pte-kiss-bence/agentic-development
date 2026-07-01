---
name: pte-openspec-dependency-graph
description: Render a Mermaid dependency graph from the backlog's epics and story cards. Use when the user wants to visualize epic/story dependencies, refresh DEPENDENCY_GRAPH.md, or check the backlog for dependency cycles.
---

The planning chain leaves every epic and story card owning its outbound edges in a `## Dependency Edges` block. This skill renders them as a single Mermaid dependency graph so a team can *see* the backlog's shape — what blocks what, where an epic leans on another, which work is a leaf.

`derived` is the governing word: the graph is a **read-only view**, never a source. It authors no epic, no story, and no edge — it reads the `## Dependency Edges` blocks and draws them. Fix a wrong arrow by editing the card's block, then re-render; never hand-edit the graph. Because it only reads, it is safe to run any time after `pte-openspec-to-stories` and is independent of `pte-openspec-bdd-tests` and `pte-openspec-jira-sync`.

Shared pipeline conventions — the *Dependency Edges* edge contract (the tag grammar, inverse-dedup, dangling-target rule), output language, and source-spec resolution — live in [`../pte-openspec-shared/CONVENTIONS.md`](../pte-openspec-shared/CONVENTIONS.md); read the *Dependency Edges* section before rendering. A complete worked render is in [`EXAMPLE.md`](EXAMPLE.md).

## Input — the only thing it reads

The `## Dependency Edges` block (inside the pipeline-only fence) of every `openspec/backlog/epics/EPIC-*.md` and `openspec/backlog/stories/**/STORY-*.md`. Nothing else on the card is read for edges — not `## Risks and Dependencies`, not prose. If no card carries a `## Dependency Edges` block, stop and say so (offer `pte-openspec-to-epics` / `pte-openspec-to-stories`, which author it) rather than drawing an empty graph.

Each bullet parses to a directed edge per the contract:
- `blocks: \`Y\`` on card X → edge `X → Y`.
- `depends-on: \`Z\`` on card X → edge `Z → X`.
- `relates-to: \`W\`` on card X → a non-directional relation `X …… W`.
- `external: <text>` on card X → a dashed leaf node for the external thing, drawn as a blocker of X (`EXT → X`).

Normalize and **dedupe inverse pairs** into one edge (X `depends-on` Y and Y `blocks` X are one `Y → X`). A `blocks`/`depends-on`/`relates-to` target that is **not** a card in the backlog is **dangling** → warn and skip it (never draw a fabricated node).

## The graph it emits

A Mermaid `flowchart LR` — left-to-right reads as execution order. Two tiers in one graph:

- **Epic = `subgraph`**, its stories the nodes inside it. Story node id is its trace ID; label is the card title.
- **An edge whose endpoint is an epic** attaches to that epic's `subgraph` id.
- **`external:` nodes** sit outside every subgraph, one per distinct external label.

**Arrow direction is fixed: blocker → blocked** — every arrow reads "must be done before". State this in an on-graph legend.

Styling (high-contrast, per `design-doc-mermaid`):
- `blocks`/`depends-on` → solid arrow `-->`.
- `relates-to` → dotted link `-.->`.
- `external` → the external node gets a dashed `classDef`.
- **Cross-epic edges are highlighted** (a `linkStyle`/class the legend names) — they are the graph's most load-bearing information: where one epic's slice waits on another's.

**Cycle detection — the graph MUST be a DAG.** Build the directed graph from the `blocks`/`depends-on` edges only (`relates-to` is non-directional, excluded). If a cycle exists it is a **planning error**, not a drawing choice: emit a prominent `## ⚠ Körfüggőség` warning listing the exact cycle path (`STORY-a → STORY-b → STORY-a`), and mark those edges red in the graph so the loop is visible. Do not silently render a cyclic graph as if it were fine.

**Overview fallback.** When a single epic's subgraph is too large or dense to read (roughly >15 story nodes, or a near-complete tangle), fall back to an **epic-level overview** for that epic — collapse its stories, draw only the epic node and its aggregated cross-epic edges. Say in the artifact that you collapsed it and why.

## Output artifact

`openspec/backlog/DEPENDENCY_GRAPH.md` — a Markdown file: a one-line intro, the embedded ` ```mermaid ` flowchart, the legend, and a `## Notes` section listing any cycles, dangling targets skipped, and collapsed epics. Diff before overwriting — if the file exists, replace it wholesale (it is fully derived, so a clean regenerate is correct), but report what changed.

**Image export is opt-in.** Only when the user asks for an image, run `design-doc-mermaid`'s `scripts/mermaid_to_image.py` to render the embedded diagram to `openspec/backlog/DEPENDENCY_GRAPH.svg`. By default emit only the Markdown.

## The two mermaid skills — roles

This skill leans on both installed mermaid skills; use each for its job rather than reinventing:
- **`mermaid-diagrams`** — the authoritative `flowchart` syntax (nodes, `subgraph`, link types, `classDef`/`linkStyle`). Consult its `references/flowcharts.md` when composing the diagram so the syntax is correct and renders.
- **`design-doc-mermaid`** — its `scripts/` Python utilities (`mermaid_to_image.py` for the opt-in `.svg` export; validation), plus its high-contrast styling and file-organization conventions.

## Steps

Copy this checklist and tick each item — the verify step is exhaustive, not a glance:

```
- [ ] 1. Scope fixed (whole backlog default, or one epic from the argument); every card's ## Dependency Edges block read
- [ ] 2. Edges parsed + normalized + inverse-deduped; dangling targets flagged; external nodes collected
- [ ] 3. DAG built from blocks/depends-on; cycles detected and listed (or none)
- [ ] 4. flowchart composed — epic subgraphs, story nodes, cross-epic edges highlighted, legend, direction blocker→blocked
- [ ] 5. DEPENDENCY_GRAPH.md written (embedded mermaid + legend + ## Notes for cycles/dangling/collapsed); image only if asked
- [ ] 6. Verified: every non-dangling edge drawn once; every cycle reported; no fabricated node; graph reads blocker→blocked
```

1. **Fix scope and read the edges.** Whole backlog by default; if the argument names an epic, scope to that epic and its stories. Read the `## Dependency Edges` block from every in-scope card (default `openspec/backlog/`, configurable — see *Output location* in `CONVENTIONS.md`). If none exists, stop and offer the authoring skills. Completion: every in-scope card's block is in hand.

2. **Parse, normalize, dedupe.** Turn each bullet into a directed edge (or a relation, or an external leaf) per the contract; normalize `depends-on`/`blocks` to one direction and dedupe inverse pairs; collect distinct `external:` labels. Flag any `blocks`/`depends-on`/`relates-to` target not present in the backlog as **dangling**. Completion (exhaustive): every bullet is either an edge, a relation, an external node, or a flagged-and-skipped dangling target — none dropped silently.

3. **Detect cycles.** Build the directed graph from the `blocks`/`depends-on` edges (exclude `relates-to`) and find every cycle. Completion: the cycle set is known — empty, or each cycle's exact path recorded.

4. **Compose the flowchart.** `flowchart LR`; one `subgraph` per epic holding its story nodes; edges by trace ID (epic-targeted edges attach to the subgraph id); cross-epic edges highlighted; `relates-to` dotted; external nodes dashed and outside subgraphs; cycle edges red; a legend stating the arrow direction and the styles. Collapse an over-large epic to its overview (noting it). Completion: the diagram covers every parsed edge and node exactly once, and is valid Mermaid (validate via `design-doc-mermaid` if unsure).

5. **Write the artifact.** `openspec/backlog/DEPENDENCY_GRAPH.md` — intro, embedded diagram, legend, and a `## Notes` section listing cycles, skipped dangling targets, and collapsed epics. Regenerate wholesale; report what changed. Export the `.svg` only if the user asked. Completion: the file exists and its `## Notes` names every anomaly.

6. **Verify exhaustively.** Every non-dangling edge appears exactly once (no inverse duplicates); every cycle is in `## Notes` and marked red; no node exists that no card declared; the arrow direction is blocker→blocked throughout; every external label is a leaf. Report any card whose block you could not parse rather than guessing an edge.
