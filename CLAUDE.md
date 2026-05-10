# CLAUDE.md

Context for agent sessions (Claude Code or Codex) working on this template repo. This is design rationale and structure — not user-facing docs (those are in `README.md` and `docs/`) and not marketplace-authoring procedures (those live in `plugins/marketplace-admin/skills/marketplace-manager/SKILL.md`).

`AGENTS.md` is a one-line `@CLAUDE.md` import — Codex auto-resolves it, so this file is read by both runtimes. See [docs/adr/0001-parallel-manifests-shared-skills-tree.md](docs/adr/0001-parallel-manifests-shared-skills-tree.md) and [CONTEXT.md](CONTEXT.md) for the resolved domain language.

## What this repo is

A GitHub template repo that, when forked, becomes a private skills marketplace for a small team — usable from both Claude Code and Codex CLI. The owner forks → runs init → pushes → teammates install via `/plugin marketplace add` (Claude Code) or `codex plugin marketplace add` (Codex). Built on each runtime's native plugin marketplace primitive — no custom services, no extra infrastructure.

Target user: business owners and small-team operators who already use Claude Code or Codex individually and want their team to share skills without copy-pasting them in Slack.

## Structure

```
.claude-plugin/marketplace.json       # Claude Code marketplace registry
.agents/plugins/marketplace.json      # Codex marketplace registry
plugins/
  team-skills/                        # the plugin teammates install
    .claude-plugin/plugin.json
    .codex-plugin/plugin.json
    skills/example-skill/             # ships as a smoke-test; owner deletes after first install works
  marketplace-admin/                  # owner-only plugin; not in teammate install instructions
    .claude-plugin/plugin.json
    .codex-plugin/plugin.json
    skills/marketplace-manager/
      SKILL.md                        # init + import + add-plugin + publish + status (all flows inline)
CLAUDE.md                             # this file; auto-loaded by Claude Code
AGENTS.md                             # @CLAUDE.md; auto-loaded by Codex
CONTEXT.md                            # domain glossary
docs/
  adr/                                # architecture decision records
  install-claude-code.md              # teammate install guide (Claude Code)
  install-codex.md                    # teammate install guide (Codex)
README.md                             # marketing pre-init; init swaps it with README.instance.md
README.instance.md                    # post-init README skeleton with REPLACE_ME_* sentinels
LICENSE                               # MIT (init can switch to UNLICENSED or other)
.gitignore
```

## Key design decisions

### 1. Native marketplaces (both runtimes), not a custom layer
Both `/plugin marketplace add` (Claude Code) and `codex plugin marketplace add` (Codex) already solve discovery, install, update, enable/disable. Anything custom would re-implement what each vendor already ships. The "10 minutes" promise is only credible because we don't build infrastructure. We ship parallel hand-maintained manifests for both runtimes; see ADR-0001.

### 2. Two plugins out of the box: `team-skills` + `marketplace-admin`
- `team-skills` is what teammates install. Skills shared across the team go here.
- `marketplace-admin` is owner-only. It contains the `marketplace-manager` skill — the tooling the owner uses to administer the marketplace.

The owner installs `marketplace-admin` from their *own* marketplace (in whichever runtime they use — Claude Code, Codex, or both). This makes the management skill globally available — owner can trigger it from any session anywhere on disk, not just from inside the cloned repo.

We deliberately do **not** ship a project-scoped `.claude/skills/` skill at repo root. That would duplicate the marketplace-admin plugin and create "which one fires" ambiguity. The plugin is the single source of truth.

### 3. Single-plugin default, multi-plugin extensible
Day-zero config has one team-facing plugin (`team-skills`). When the team grows enough to want domain separation (`sales-skills`, `ops-skills`), the marketplace-manager skill's `add-plugin` flow scaffolds new plugins. We chose this default because:
- Single install command is honest about "10 minutes."
- Skills load progressively (only when triggered) — there's no per-skill cost to bundling many in one plugin.
- Splitting later is a directory move + marketplace.json edit, not a migration users feel.

### 4. Marketplace-manager is one SKILL.md, all flows inline
Five flows (init, import, add-plugin, publish, status) live in a single SKILL.md, not split across sub-files. The user explicitly chose this over progressive disclosure. Trade-off: every trigger pays the full token cost; in exchange, simpler authoring and one place to read.

### 5. Sentinels: `REPLACE_ME_*`
First-run detection uses placeholder strings (`REPLACE_ME_company_name`, `REPLACE_ME_marketplace_name`, `REPLACE_ME_github_owner/REPLACE_ME_github_repo`) embedded across the dual-runtime manifest set. Init must rewrite all of them. Sentinels currently appear in:

- `.claude-plugin/marketplace.json`
- `.agents/plugins/marketplace.json`
- `plugins/team-skills/.claude-plugin/plugin.json`
- `plugins/team-skills/.codex-plugin/plugin.json`
- `plugins/marketplace-admin/.claude-plugin/plugin.json` (description line)
- `plugins/marketplace-admin/.codex-plugin/plugin.json` (description line)
- `README.instance.md`
- `LICENSE`

Loud and unmistakable beats polished-but-ambiguous. The marketplace name itself ships with a *working default* (`team-marketplace`) rather than a sentinel, because the owner needs to install `marketplace-admin@team-marketplace` *before* they can run init. Marketplace name is sentinel-free; init may rename it.

### 6. README swap pattern
- `README.md` (pre-init): marketing copy aimed at someone evaluating "should I use this template?"
- `README.instance.md`: skeleton for the post-init README, written for the team that will use this specific instance.
- Init deletes the marketing README and renames `README.instance.md` → `README.md`. The marketing copy is unrecoverable post-init by design — it was for a different audience and dates badly.

### 7. Import flow: copy, not move
When the owner imports a skill from `~/.claude/skills/<name>/` into the marketplace, we *copy* by default rather than move or symlink. Copy matches the "I'm publishing this" mental model. The skill then offers to clean up the personal copy as an opt-in second step (after the owner installs the team plugin themselves), which gets to single-source-of-truth without surprising them.

### 8. Update flow: auto-update (Claude Code) / explicit upgrade (Codex)
Claude Code teammates toggle "Enable auto-update" in the `/plugin` UI once during install — new skills land on next startup. Codex teammates run `codex plugin marketplace upgrade <name>` explicitly (Codex has no auto-update toggle as of writing). Both install docs document the relevant pattern. Manual `/plugin marketplace update` is the fallback for Claude Code.

### 9. Owner config persistence
The `marketplace-manager` skill stores `repo_path` at `~/.config/claude-marketplace-admin/config.json` so it can find the marketplace clone when invoked from any cwd. Single field, single marketplace per machine. Multi-marketplace support is a v2 concern.

### 10. Dual-runtime: parallel manifests, single shared skills tree
Codex CLI ships a near-identical `plugin marketplace add` primitive, with `marketplace.json` at `.agents/plugins/marketplace.json` and per-plugin `.codex-plugin/plugin.json` files. SKILL.md format is byte-for-byte identical across runtimes. We track parallel hand-maintained manifests for both; the `marketplace-manager` skill writes both sets in lockstep. Skills tree is shared. `AGENTS.md` is a one-line `@CLAUDE.md` import so this file serves both runtimes. See ADR-0001 for the trade-offs against generation-based approaches.

## Things deliberately NOT included

These were considered and dropped:

- **CI workflow validating marketplace.json** — judged not worth the complexity for v1.
- **PreToolUse validate hook on edits** — adds noise; the skill already enforces structure on creation.
- **Slash commands (`/init`, `/add-skill`)** — duplicates the marketplace-manager skill. Skill triggers via natural language and works from anywhere.
- **A "plugin authoring" skill teaching how to write skills/commands/hooks** — out of scope. The built-in `write-a-skill` / `skill-creator` handles authoring; this template handles *sharing* finished skills.
- **Starter pack of curated skills** — generic skills are noise on day one; opinionated ones are wrong for most teams. We ship one trivial example skill as a smoke test, nothing more.
- **A separate authoring skill (one for init, one for ongoing)** — single skill with mode dispatch is simpler.

## When working on this repo

- **Marketplace-authoring procedures (init, import, etc.) belong in `SKILL.md`, not here.** This file is design rationale; the skill is the runtime source of truth.
- **If you change the sentinel strings, update every location** listed in §5 above. The init flow in SKILL.md must also be updated.
- **If you add a new flow to the marketplace-manager skill**, update its `description` frontmatter — that's how both Claude Code and Codex decide when to trigger it.
- **The `team-marketplace` default name is load-bearing** — it's referenced in the pre-init README as the name to install `marketplace-admin@` from for both runtimes. Don't rename it without updating the README.
- **Any change to `.claude-plugin/marketplace.json` requires a mirrored change to `.agents/plugins/marketplace.json`,** and any change to a plugin's `.claude-plugin/plugin.json` requires a mirrored change to its `.codex-plugin/plugin.json`. The skill is the only writer in normal operation; manual edits must touch both files.
