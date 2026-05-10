# CONTEXT

Domain language for this template repo. Update inline as terms are resolved.

## Core concepts

- **Marketplace** — a Git repository that publishes one or more **Plugins** for installation by an agent CLI. Both Claude Code and Codex CLI implement this primitive natively.
- **Plugin** — an installable bundle of **Skills** (and optionally hooks, MCP servers, etc.) that a teammate enables with one install command. Identified by a kebab-case `name`.
- **Skill** — a `SKILL.md` file (plus optional bundled `scripts/`, `references/`, `assets/`) with YAML frontmatter `name` and `description`. The `description` is the **triggering surface**: the agent reads it and decides when to load the body. Bodies are lazily loaded — they don't cost context until triggered. Identical format across Claude Code and Codex.
- **Owner** — the person who forks the template, runs init, and administers the marketplace. Operates `marketplace-admin`.
- **Teammate** — a member of the owner's team who installs the marketplace (read-only consumer). Runs `team-skills` (and any future team-facing plugins).

## Manifest files

Two parallel sets per supported ecosystem; skills tree is shared.

| File | Claude Code path | Codex path |
|---|---|---|
| Marketplace registry (repo root) | `.claude-plugin/marketplace.json` | `.agents/plugins/marketplace.json` |
| Plugin manifest (per plugin) | `plugins/<name>/.claude-plugin/plugin.json` | `plugins/<name>/.codex-plugin/plugin.json` |
| Skills tree | `plugins/<name>/skills/` (shared) | `plugins/<name>/skills/` (shared) |
| Per-directory instructions | `CLAUDE.md` | `AGENTS.md` |

The skills tree is single-source-of-truth — `SKILL.md` format is identical, and both runtimes lazy-load bodies on description match.

## Codex-specific schema requirements

Codex `marketplace.json` requires fields Claude Code doesn't:

- `policy.installation` — `AVAILABLE` (default) | `INSTALLED_BY_DEFAULT` | `NOT_AVAILABLE`
- `policy.authentication` — `ON_INSTALL` (default) | `ON_USE`
- `category` — display bucket (e.g., `Productivity`, `Development`)
- `source` is an object `{"source":"local","path":"./plugins/<name>"}`, not a string

Defaults the template will set unless overridden: `installation: AVAILABLE`, `authentication: ON_INSTALL`, `category: Productivity`.

## Sentinels

Pre-init placeholders that init must rewrite:

- `REPLACE_ME_company_name`
- `REPLACE_ME_marketplace_name` (only in `README.instance.md` — the marketplace name itself ships with working default `team-marketplace`)
- `REPLACE_ME_github_owner` / `REPLACE_ME_github_repo`

Locations: `.claude-plugin/marketplace.json`, `.agents/plugins/marketplace.json` (post-v2), each plugin's `plugin.json` (both flavors post-v2), `README.instance.md`, `LICENSE`.

## What we deliberately do NOT call things

- "Extension" — Codex docs sometimes use this informally; we use **Plugin** to match both CLIs' canonical command names.
- "Agent" — overloaded. The runtime is the **agent CLI** (Claude Code, Codex CLI). We never call a Skill an "agent."

## Repo-level instruction files

- `CLAUDE.md` — auto-loaded by Claude Code; full content lives here.
- `AGENTS.md` — auto-loaded by Codex; single line `@CLAUDE.md` (Codex resolves the import). Avoids duplication and drift. See ADR-0001.

## Resolved design decisions (cross-ref to ADRs)

- **Both runtimes are first-class peers.** READMEs and install docs show Claude Code and Codex commands side-by-side. The marketplace-manager skill never asks "which runtime?" — it always writes both manifest sets.
- **Init auto-defaults Codex-specific fields.** No extra prompts at init. `policy.installation: AVAILABLE`, `policy.authentication: ON_INSTALL`, `category: Productivity`. No `interface` block on init; owner edits later if they ever publish externally.
- **Skills tree is single-source-of-truth.** `plugins/<name>/skills/` is read by both runtimes — `SKILL.md` format is identical and bodies lazy-load on description match in both.
