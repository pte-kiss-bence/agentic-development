# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

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

## Secrets

`.devcontainer/.env` is git-ignored and injected into the container via `runArgs: --env-file`. Keys (template in `.env.example`): `GIT_USER_NAME`, `GIT_USER_EMAIL`, `GITHUB_PERSONAL_ACCESS_TOKEN` (GitHub HTTPS auth + GitHub MCP), `CONTEXT7_API_KEY` (optional — context7 also works keyless). Values must be unquoted and use full-line comments only (`docker --env-file` parsing).
