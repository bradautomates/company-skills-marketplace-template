# Company Skills Marketplace

A GitHub template for spinning up a **private skills marketplace** for your team in under 10 minutes. Works with both [Claude Code](https://claude.ai/code) and [Codex CLI](https://github.com/openai/codex).

> **Stop copy-pasting skills around in Slack.** If your team uses Claude Code or Codex individually, this template gives you a shared execution layer: one private repo, one install command per teammate, and skills that update across the team automatically.

## What this is

Both Claude Code and Codex support private plugin marketplaces natively, with near-identical primitives. This template wires up both with sensible defaults so you don't have to read either spec:

- **Native marketplaces, both runtimes.** Uses `.claude-plugin/marketplace.json` (Claude Code) and `.agents/plugins/marketplace.json` (Codex) side-by-side. No extra services to run.
- **One shared `skills/` tree.** SKILL.md format is identical across runtimes, so your team writes a skill once and both ecosystems load it.
- **Two plugins out of the box.** `team-skills` is where your team's skills live (one example skill ships so you can confirm install works). `marketplace-admin` ships an owner-only `marketplace-manager` skill that handles setup, importing skills, publishing, and status — usable from any session, anywhere on disk.
- **Private by default.** Designed assuming you fork private. Auth via your team's existing GitHub access.
- **Auto-update.** Teammates enable auto-update once; new skills land on next startup.

## Who this is for

Business owners and small-team operators who already use Claude Code or Codex individually and want their team to share the same execution layer — without rebuilding skills from scratch or copy-pasting them in Slack.

## How it works

1. **Click "Use this template" → set Private → clone the new repo locally.**

2. **Make sure `gh` is authenticated** (`gh auth status`; if not, `gh auth login`).

3. **Install the admin plugin from your runtime of choice:**

   **Claude Code:**
   ```
   /plugin marketplace add <your-github-username>/<your-repo-name>
   /plugin install marketplace-admin@team-marketplace
   ```

   **Codex CLI:**
   ```
   codex plugin marketplace add <your-github-username>/<your-repo-name>
   codex plugin install marketplace-admin@team-marketplace
   ```

4. **Trigger the marketplace-manager skill** ("set up the marketplace"). It collects your company name, marketplace name, and a license preference, then rewrites the configuration in both runtimes' manifests, swaps the README, commits, and pushes.

5. **Share the install docs with your team.** Claude Code teammates: [`docs/install-claude-code.md`](docs/install-claude-code.md). Codex teammates: [`docs/install-codex.md`](docs/install-codex.md). They each run two commands and toggle auto-update once.

6. **From then on, anywhere on your machine,** trigger the `marketplace-manager` skill to import existing skills (e.g., from `~/.claude/skills/` or `~/.codex/skills/`) into the marketplace, scaffold new plugins as you grow, and publish. Both runtimes' manifests stay in sync automatically.

## What's in the box

```
.claude-plugin/marketplace.json       # Claude Code marketplace registry
.agents/plugins/marketplace.json      # Codex marketplace registry
plugins/
  team-skills/                        # where your team's skills live
    .claude-plugin/plugin.json
    .codex-plugin/plugin.json
    skills/example-skill/             # delete after first install
  marketplace-admin/                  # owner-only management plugin
    .claude-plugin/plugin.json
    .codex-plugin/plugin.json
    skills/marketplace-manager/       # init, import, add-plugin, publish, status
CLAUDE.md                             # design rationale (auto-loaded by Claude Code)
AGENTS.md                             # @CLAUDE.md (auto-loaded by Codex)
CONTEXT.md                            # domain glossary
docs/
  adr/                                # architecture decisions
  install-claude-code.md              # teammate install guide (Claude Code)
  install-codex.md                    # teammate install guide (Codex)
README.md                             # this file (replaced by instance README on init)
README.instance.md                    # post-init README (with sentinels)
LICENSE                               # MIT (changeable during init)
.gitignore
```

## Prerequisites

- [Claude Code](https://claude.ai/code) **or** [Codex CLI](https://github.com/openai/codex) (or both) installed
- [`gh` CLI](https://cli.github.com/) authenticated (`gh auth login`) — used for private repo access and visibility checks
- A GitHub account with permission to create private repos

## About

Built by [Brad Bonanno](https://www.youtube.com/@bradbonanno) at [Solaris Automation](https://www.solarisautomation.io/) — we help teams ship AI-powered workflows without rebuilding infra. Subscribe on [YouTube](https://www.youtube.com/@bradbonanno) for walkthroughs of templates like this one.

## License

MIT in this template (replace during init if you want UNLICENSED for proprietary internal use).
