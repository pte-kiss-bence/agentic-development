Author a new Claude Code skill that converts OpenSpec specifications into Agile Epics (Atlassian/SAFe-style). This repo is @pte/agentic-development.

## Before writing anything
1. Read .claude/skills/writing-great-skills/SKILL.md and its GLOSSARY.md in full. Apply its principles to the skill you author: predictability first, leading words, checkable completion criteria, progressive disclosure (push reference behind context pointers), single source of truth, no-op pruning.
2. Read openspec/config.yaml and one existing sibling skill (e.g. .claude/skills/openspec-propose/) to match house structure and frontmatter conventions.

## Deliverable
Create .claude/skills/pte-openspec-to-epics/SKILL.md (plus disclosed reference files only if SKILL.md would otherwise sprawl).

- Invocation: MODEL-INVOKED. Omit disable-model-invocation. Write a model-facing description: lead with the action, one trigger per branch (e.g. "Use when the user wants Agile Epics from OpenSpec specs, mentions epics/backlog from a spec or change delta, or wants specs turned into a product backlog"). No duplicated synonyms.
- The skill reads OpenSpec specs from BOTH openspec/specs/** (main specs) and a change's delta specs openspec/changes/<id>/specs/**/spec.md. OpenSpec spec format is `### Requirement` headers containing `#### Scenario:` blocks with WHEN/THEN/AND lines — the skill must parse these.
- The skill WRITES one markdown Epic per file into an output directory (default epics/, configurable), one Epic per capability or coherent requirement group. Filenames slugged from the epic title.

## Domain knowledge to bake into the skill
"Epic" is NOT defined in the official Scrum Guide (which defines Product Backlog Items); authoritative Epic best practice comes from Atlassian and SAFe. Encode these rules:
- An Epic is a large body of work sized weeks-to-months, split into 5-15 user stories. If it would exceed ~15 stories, instruct splitting into multiple Epics. Split stories by user value / journey, NOT by technical layer.
- Keep Epics lean and strategic — a guide to value, not a task bucket.
- Each Epic states a hypothesis in the form: "Ha [X], akkor [Y] a [Z] csoportnak, mérve [W]-vel."

Epic file anatomy (sections in this order). Apply OpenSpec→Epic mapping:
- Cím — epic name (from capability / requirement-group theme)
- Háttér és kontextus — narrative from the spec's purpose/overview
- Probléma / lehetőség — the problem the spec solves
- Hipotézis — the If/Then/measured-by statement above
- Hatókör és nem-célok — in-scope requirements + explicit non-goals
- Érintettek — stakeholders / affected roles
- Sikermutatók — measurable success metrics
- Magas szintű elfogadási kritériumok — derived from the requirements' Scenarios
- User story-k — 5-15 stories, each "Mint [szerep], szeretnék [cél], hogy [érték]", with acceptance criteria mapped from the matching `#### Scenario:` WHEN/THEN
- Függőségek és kockázatok
- Forrás-spec hivatkozás — traceability map (see Trace ID convention below): a markdown table with columns `Story ID | Epic ID | Forrás ### Requirement | Lefedett #### Scenario-k`, one row per story, listing the exact source Requirement name(s) and the verbatim Scenario titles each story covers. This table is the contract the BDD skill consumes — keep Requirement and Scenario titles verbatim.

## Trace ID convention (this skill OWNS it; pte-openspec-bdd-tests consumes it)
The epic skill ASSIGNS stable IDs; downstream skills only reference them. Grammar:
- `EPIC-<epic-slug>` — `epic-slug` = kebab-case of the epic title (capability / requirement-group theme).
- `STORY-<epic-slug>-<requirement-slug>` — `requirement-slug` = kebab-case of the story's primary source `### Requirement` name. Anchored on the requirement, NOT on a running index, so the ID is stable when stories are reordered.
- When several stories split the same primary requirement (split by user value/journey), append a stable `-<aspect-slug>` derived from the story's value/goal (e.g. `STORY-checkout-fizetes-utalas`, `STORY-checkout-fizetes-kartya`) — never a positional number. The aspect-slug describes the slice, so it survives reordering too.
Rules:
- Story decomposition is a judgement call (grouping/splitting requirements), so the requirement→story split is NOT derivable from the spec alone — that is exactly why the epic emits the traceability map above, and why BDD must read it rather than re-derive. (The ID grammar is stable; the mapping is what BDD needs.)
- Put `EPIC-<epic-slug>` in the Cím section and the `STORY-…` ID at the start of each story under "User story-k".
- Keep each STORY ID unique within its epic; if two slices would collide, the `-<aspect-slug>` is what disambiguates them.
- IDs and slugs are kept verbatim (not translated), even though surrounding content is Hungarian.

## Output language rule (critical — put it near the top of the skill body)
All Epic file CONTENT is written in HUNGARIAN, matching openspec/config.yaml's artifact convention. Keep verbatim (do NOT translate): code, identifiers, API names, CLI commands, file paths, requirement names, and commit-type keywords (feat/fix/...). The section headings above are already Hungarian — use them as-is.

## Completion criteria for the skill's own steps
Make each step end on a checkable, exhaustive criterion — e.g. "every `### Requirement` in the source spec is accounted for in exactly one Epic or explicitly listed as out of scope", not "produce some epics".

## Scope locks
- Only create/edit files under .claude/skills/pte-openspec-to-epics/.
- Do NOT modify openspec/specs/**, openspec/config.yaml, CLAUDE.md, or any other skill.
- Do NOT add dependencies or scripts.
- Only make changes directly requested. No extra abstractions, no refactoring of sibling skills.

## Stop conditions
- After writing SKILL.md, STOP and show me the full file for review before creating any further reference files.
- Ask before overwriting any existing file.

## Done when
.claude/skills/pte-openspec-to-epics/SKILL.md exists, is model-invoked with a trigger-rich description, encodes the Epic anatomy + best-practice rules + Hungarian output rule + OpenSpec parsing, and every step has a checkable completion criterion.
