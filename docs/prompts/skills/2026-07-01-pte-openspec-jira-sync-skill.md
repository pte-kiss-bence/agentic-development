# Prompt — pte-openspec-jira-sync skill authoring

- **Target tool:** Claude Code `/writing-great-skills`
- **Generated:** 2026-07-01
- **Builds:** `pte-skills/pte-openspec-jira-sync/` skill + Atlassian Remote MCP wiring
- **Language:** Prompt is English (highest instruction-following fidelity for agentic Claude Code). Skill artifact content stays Hungarian, enforced inside the prompt.

---

```
Author a new Claude Code skill named `pte-openspec-jira-sync`, and wire up the Atlassian Remote MCP server so the skill can run. Follow the repo's existing conventions — READ them first, do not guess.

## Starting state (repo context, carry forward)
- This is the `@pte/agentic-development` repo: a shared agentic-development process. Under `pte-skills/` a linear planning chain exists:
  1. `pte-openspec-to-epics` → `epics/<epic-slug>.md` (mints `EPIC-…` / `STORY-…` trace IDs + a "Forrás-spec hivatkozás" traceability table)
  2. `pte-openspec-to-stories` → `stories/<epic-slug>/<story-slug>.md` (story cards; `STORY-…` + `EPIC-…` IDs in the header, acceptance criteria, a BDD test section)
  3. `pte-openspec-bdd-tests` → fills each story card's `## BDD teszt` section in place with a fenced ```gherkin block tagged `@EPIC-…` `@STORY-…`
  4. (build) `pte-openspec-tdd-execute` — independent, does not touch this chain
- All artifact CONTENT is Hungarian; code identifiers (EPIC-…, STORY-…, filenames, API names) stay verbatim and are never translated. Keep the new skill's artifact content Hungarian too, matching its siblings.
- No Atlassian/Jira MCP is wired anywhere in the repo yet.
- Sibling skill layout: a directory under `pte-skills/`, containing `SKILL.md` (YAML frontmatter: `name`, `description`) + an `EXAMPLE.md` with a worked end-to-end example. `pte-openspec/SKILL.md` is the chain's router.

## Target state
Two deliverables:
A) `pte-skills/pte-openspec-jira-sync/` — a new local skill (a sibling, NOT a separately packaged plugin), positioned AFTER chain step 3 (bdd-tests).
B) The Atlassian Remote MCP wired per the repo's config conventions, so the skill's Jira tools are available inside the container.

## A) Skill behavior — precise spec
Input: `epics/*.md` and `stories/<epic-slug>/*.md` with their trace IDs.
Mapping to Jira:
- Epic md → Jira issue type **Epic**
- Story md → Jira issue type **Story**, with the `parent` field linking the story to its epic's Jira key (epic→story hierarchy)

### Field-ownership model (this is the sustainable sync methodology — build the skill on THIS)
Not "direction by existence", but one owner PER FIELD:

| Field | Owner | Direction |
|-------|-------|-----------|
| Summary (title), Description (user story + context), acceptance criteria, BDD (Gherkin), labels (trace ID), issue type, epic↔story `parent` | Local | **Local → Jira** (push, overwrite) |
| Status, assignee, sprint, estimate (when the team sets it), comments, Jira key/URL | Jira | **Jira → Local** (pull into a `## Jira szinkron` block in the story card) |

This makes re-runs idempotent: each field has exactly one owner, and the sync never clobbers the other side's work.

### Matching / idempotency
- Every Jira issue gets a `trace:STORY-…` or `trace:EPIC-…` label.
- Resolve in order: (1) a Jira key recorded in the story/epic card; if none → (2) `searchJiraIssuesUsingJql` by label (`labels = "trace:STORY-…"`); if no hit → (3) `createJiraIssue`.
- After creation, write the returned Jira key + URL + status back into the card's `## Jira szinkron` section (key, issue type, status, URL, last-synced timestamp). Create the section if missing; do NOT touch the card's other (locally-owned) content.

### cloudId + project resolution
- First `getAccessibleAtlassianResources` (then `getVisibleJiraProjects` as needed) for the `cloudId` — every Jira tool call requires it.
- Get the target Jira project key by asking the user (AskUserQuestion) or reading a simple config/card header; do not hardcode it.

### Atlassian MCP tools to use (exact names)
Read: `getAccessibleAtlassianResources`, `getVisibleJiraProjects`, `getJiraProjectIssueTypesMetadata`, `getJiraIssue`, `lookupJiraAccountId`, `searchJiraIssuesUsingJql`, `getTransitionsForJiraIssue`.
Write: `createJiraIssue`, `editJiraIssue`, `addCommentToJiraIssue`, `transitionJiraIssue`.

### Run safety
- Default to **dry-run**: first print a summary plan (what it would create / update / pull back), and **ask for confirmation before ANY Jira write (`createJiraIssue`/`editJiraIssue`/`transition`)**.
- At the end of a run, print a short report: created / updated / pulled issues as trace ID → Jira key pairs.

## B) Wiring the Atlassian MCP
- Inspect HOW the existing MCP-providing plugins (`context7`, `playwright`) are wired in `.claude/settings.json` and in the devcontainer scripts, and mirror that pattern.
- Wire the Atlassian Remote MCP server: `https://mcp.atlassian.com/v1/mcp` (OAuth 2.1 or API-token auth). For a container/headless environment, env-based (API-token) auth is the reproducible choice — follow the repo's Secrets convention: values go in `.devcontainer/.env`, injected via `--env-file`; add keys to `.env.example` with full-line comments; **NEVER commit a secret/token**.
- Every config mutation must be **idempotent** (grep-guard before append, merge-don't-overwrite for JSON), per the devcontainer rules in `CLAUDE.md`. If you modify a script, use the shared logger (`log.sh`).

## Repo conventions to follow
- `SKILL.md` frontmatter in the sibling style (`name: pte-openspec-jira-sync`, `description:` with clear "use when…" triggers). Create an `EXAMPLE.md` with a worked example.
- Hook it into the chain: after `pte-openspec-bdd-tests`, offer this skill ("offer next step" convention), and update the `pte-openspec/SKILL.md` router with the new step.
- Artifact content is Hungarian; trace IDs and Jira keys stay verbatim.

## Forbidden actions / stop conditions
- ONLY the requested scope. Do not refactor the sibling skills, do not rewrite untouched files, do not add gratuitous abstraction or dependencies.
- Stop and ask BEFORE: any Jira write on the first real run; adding a dependency; modifying a devcontainer script (`post-create.sh`, etc.); deleting any file.
- Do NOT overwrite locally-owned content from Jira, and do NOT overwrite Jira workflow fields (status/assignee/sprint) from local.
- Never commit a secret/token; only env references may go into committed files.

When done: a short summary of the files created, how the wiring was done, and the user-side to-dos (which env keys to fill in, how the Atlassian MCP authenticates).
```

🎯 **Target:** Claude Code `/writing-great-skills` — English agentic skill-authoring prompt with exact Atlassian MCP tool names, exact pipeline artifact paths, a field-ownership sync model (idempotent, no clobber), and scope/stop-condition locks. Skill artifact content is kept Hungarian by explicit instruction.

> Setup note: this drives an agentic tool with real system access (edits repo files, wires an MCP server). Review the scope locks, forbidden actions, and stop conditions before running. Confirm the `epics/` / `stories/` paths and the `context7`/`playwright` wiring pattern match the actual repo before it starts. The Atlassian MCP still needs user-side OAuth/API-token auth after wiring.
