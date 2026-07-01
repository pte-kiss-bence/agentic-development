# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Communication language

**Converse in Hungarian.** All conversational chat with the user — questions, progress, status, summaries, `AskUserQuestion` prompts — is in Hungarian by default. This is the chat channel only; it does not change what gets written into files. Existing carve-outs still hold and win where they apply: **artifact/product content stays Hungarian** anyway (epics, stories, docs, code, commit messages), but **`prompt-master` output stays English** (see the prompt-master rule below — English is the deliberate instruction-following channel for AI tools), and any content a rule already pins to a specific language keeps that language.

## What this repository is

`@pte/agentic-development` is a **template/scaffold for a shared agentic development process** reused across PTE projects — not an application. There is no app source code yet; the substance of the repo is the **devcontainer** (`.devcontainer/`) and the **Claude Code configuration** (`.claude/`). The active work (branch `feature/create-agentic-development-workflow`) is building out that workflow.

## Commands

- The package manager is **pnpm** (activated via corepack). Do not use npm or yarn.
- `pnpm install` — install project dependencies (currently none beyond the empty root package).
- **No build / lint / test scripts exist yet** — `package.json` `scripts` is `{}`. Don't assume a `pnpm test`/`pnpm build` exists; if you add one, wire it into `package.json`.
- Container-global CLIs installed by `post-create.sh` (not project dependencies, so they won't appear in `package.json`): `openspec` (`@fission-ai/openspec`, spec-driven development), `ncu` (`npm-check-updates`), `skills`.

## Devcontainer architecture

The devcontainer is the core of this repo. It is **Linux-host only** (WSL2 on Windows, or native Linux/macOS); all scripts are **POSIX `sh` (no bashisms)** so they run identically on the host and inside the container. A Windows-drive workspace (9p/drvfs) is detected and warned against — ext4/WSL2 is required for acceptable file I/O.

Lifecycle (see `devcontainer.json`):
1. `initializeCommand` → `init.sh` runs **on the host before build**: creates the per-repo Claude state dir `~/.claude-devcontainer/<repo>` (bind-mount source, pre-created so Docker doesn't own it as root) and seeds `.devcontainer/.env` from `.env.example`.
2. `postCreateCommand` → `post-create.sh` runs **in the container after creation**: installs CLI tooling, activates pnpm, installs the global CLIs above, runs `pnpm install`, writes the container-scoped Claude setting (below), applies the permission guard, and configures git (`safe.directory`, identity + GitHub PAT auth from `.env`, `core.editor=nano`).
3. `postStartCommand` → `fix-permissions.sh` runs **on every start**: heals root-owned paths and applies default POSIX ACLs so files created later (even by root processes) stay `vscode`-writable. `node_modules` is skipped; `.git` is intentionally included.

Two hard conventions when editing these scripts:
- **Idempotency is required** — every script re-runs on each rebuild/start. Guard mutations (grep checks before appending to `/etc/bash.bashrc` or `/etc/gitconfig`, `mkdir -p`, merge-don't-overwrite for `.env` and JSON settings).
- **Use the shared logger** — source `log.sh`, set `LOG_SCOPE` first, then use `log_step`/`log_info`/`log_ok`/`log_warn`/`log_error` and `log_enable_error_trap`.

## Claude Code configuration

There is a deliberate **two-tier settings split**:
- `.claude/settings.json` is **committed** and applies both on the host and in the container. It enables plugins from the `claude-plugins-official` marketplace (`claude-md-management`, `context7`, `frontend-design`, `playwright`, `ralph-loop`, `typescript-lsp`) and sets `autoMemoryEnabled: false`.
- **Container-only** settings live in `CLAUDE_CONFIG_DIR` (`/home/vscode/.claude-devcontainer`, the per-repo host bind mount) and are written by `post-create.sh` — currently `permissions.defaultMode: "bypassPermissions"`. This is kept **out** of the committed file on purpose: bypass-permissions is appropriate only inside the sandboxed container, so opening the repo on the host keeps normal approval prompting. Do not move that setting into `.claude/settings.json`.

## OpenSpec workflow

When running any OpenSpec workflow — the `/opsx:*` slash commands or the `openspec-*` skills (propose, explore, apply, archive, sync) — respond in **caveman** style (see the `caveman` skill, default `full` level) for all conversational output: progress lines, status, summaries, and AskUserQuestion prompts. Compress the chatter, keep technical substance exact. Two carve-outs:
- **Artifacts and code stay normal prose.** Contents you write to files (`proposal.md`, `design.md`, `tasks.md`, spec deltas, main `openspec/specs/**`, application code, commit messages) are never caveman.
- **Honor caveman auto-clarity.** Drop the style for irreversible-action confirmations (e.g. the archive `mv`, incomplete-task/artifact warnings) and anywhere compression risks a misread. In explore mode keep ASCII diagrams and don't let terseness cut the actual reasoning.

This rule lives here (not in the generated `.claude/commands/opsx/*` or `.claude/skills/openspec-*` files) on purpose: `openspec update` regenerates those and would wipe edits. `CLAUDE.md` is not managed by OpenSpec, so it survives updates. (`openspec/config.yaml` → `context:`/`rules:` also survives update but is artifact-content-scoped, so it's the wrong place for a chat-style rule.)

**After a propose completes, offer a right-sized next step.** Once `/opsx:propose` or the `openspec-propose` skill finishes generating a change's artifacts, always offer to continue — but the offer is a **right-sizing branch, not a yes/no**, because a change can be anything from a full feature to a small tweak or a bug fix. Use a single AskUserQuestion prompt (caveman, per the rule above) so the user opts in, with these paths:
- **Full feature (planning chain)** — the change introduces a real capability worth decomposing into a backlog. Start the `pte-openspec` pipeline by invoking `pte-openspec-to-epics` in **mint mode** (planning-chain entry point, stages 2–4: epics → story cards → BDD tests), then `pte-openspec-tdd-apply` builds it test-first once planning is ready.
- **Single story (no new epic)** — the change is bigger than a tweak but doesn't merit an epic: one (or a few) stories' worth of work. Two sub-cases, decided by whether the affected capability already has an epic:
  - *Affects an existing epic (default for a change request, since a change modifies existing behaviour):* run `pte-openspec-to-epics` in **reconcile mode** to fold the new story under the epic the delta touches (reuses the `EPIC-…`, mints the new `STORY-…`), then `pte-openspec-to-stories` → `pte-openspec-bdd-tests` → `pte-openspec-tdd-apply`.
  - *No epic covers the capability:* run `pte-openspec-to-stories` in **epic-less mode** to author the standalone card(s) straight from the delta, then `pte-openspec-bdd-tests` → `pte-openspec-tdd-apply`. Skips `pte-openspec-to-epics` entirely.
- **Small change / bug fix (skip the planning chain)** — a tweak, small refactor, or bug that touches ~one capability and carries no product decision. Epics/stories/BDD would be pure ceremony, so go straight to the build: `pte-openspec-tdd-apply` (test-first — for a bug, this means a failing test that reproduces it, then the fix), or plain `/opsx:apply` when a test adds no value. For a bug whose root cause isn't already obvious, run the `diagnosing-bugs` skill first to pin down the cause, then feed that into the test-first fix. For bugs this whole path is the default; only escalate upward if the fix turns out to be a feature in disguise.

Offer, don't auto-run, and don't force the whole chain onto small work: the propose stays the single entry point and audit trail, but the planning chain is the scaling tool, applied only when there's something to scale. This rule lives here for the same wipe-on-`openspec update` reason as the caveman rule.

## prompt-master output

Every prompt produced by the `prompt-master` skill (`/prompt-master`) must be saved to a file under `docs/prompts/<category>/<YYYY-MM-DD>-<slug>.md`:
- `<category>` — the kind of prompt (e.g. `skills`, `devcontainer`, `docs`).
- `<YYYY-MM-DD>` — date the prompt was generated.
- `<slug>` — short kebab-case name of what the prompt builds.
- The file holds the generated prompt verbatim (the copyable block), with code/identifiers kept as-is.

Save the file in addition to showing the prompt in chat; create the `<category>` directory if missing.

**Write the generated prompt in English.** The prompt is a task-planning channel for an AI tool (usually Claude Code), and English gives the highest instruction-following fidelity — even when the chat conversation is in Hungarian. This is not a translation of parity: for precise, multi-constraint agentic prompts English is measurably more reliable than Hungarian (a mid-resource language), so default to English regardless of chat language. Carve-out: any **artifact content** the prompt tells the target tool to produce (skill text, epics/stories, docs, code) stays in the project's language (Hungarian) — instruct that explicitly inside the prompt. Prompt = English; product content it specifies = Hungarian.

## Secrets

`.devcontainer/.env` is git-ignored and injected into the container via `runArgs: --env-file`. Keys (template in `.env.example`): `GIT_USER_NAME`, `GIT_USER_EMAIL`, `GITHUB_PERSONAL_ACCESS_TOKEN` (GitHub HTTPS auth + GitHub MCP), `CONTEXT7_API_KEY` (optional — context7 also works keyless). Values must be unquoted and use full-line comments only (`docker --env-file` parsing).
