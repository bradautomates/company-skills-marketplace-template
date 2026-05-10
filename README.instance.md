# REPLACE_ME_marketplace_name

Private skills marketplace for **REPLACE_ME_company_name**. Works with both Claude Code and Codex CLI.

## For teammates: install

You need [Claude Code](https://claude.ai/code) **or** [Codex](https://github.com/openai/codex), plus the `gh` CLI (`gh auth login`) authenticated to a GitHub account with read access to this repo.

```
gh auth status   # confirm you're signed in
```

### Claude Code

```
/plugin marketplace add REPLACE_ME_github_owner/REPLACE_ME_github_repo
/plugin install team-skills@REPLACE_ME_marketplace_name
```

Then in Claude Code, open `/plugin` → **Marketplaces** tab → toggle **Enable auto-update**. New skills land on your next startup.

Full guide: [docs/install-claude-code.md](docs/install-claude-code.md).

### Codex CLI

```
codex plugin marketplace add REPLACE_ME_github_owner/REPLACE_ME_github_repo
codex plugin install team-skills@REPLACE_ME_marketplace_name
```

To pull updates manually: `codex plugin marketplace upgrade REPLACE_ME_marketplace_name`.

Full guide: [docs/install-codex.md](docs/install-codex.md).

## For the owner

You manage this marketplace via the `marketplace-admin` plugin's `marketplace-manager` skill, installable from this marketplace itself in either runtime:

**Claude Code:**
```
/plugin install marketplace-admin@REPLACE_ME_marketplace_name
```

**Codex CLI:**
```
codex plugin install marketplace-admin@REPLACE_ME_marketplace_name
```

Once installed, trigger it from **anywhere** on your machine ("add a skill to the team marketplace", "publish to the marketplace", "marketplace status"). Your repo path is saved at `~/.config/claude-marketplace-admin/config.json`.

The skill handles: importing existing skills from `~/.claude/skills/` or `~/.codex/skills/` (or any path) into the marketplace, scaffolding new plugins (when you want to split by domain — `sales`, `ops`, etc.), bumping versions, committing, and pushing. Both runtimes' manifests stay in sync automatically.

## What's in the box

- **`team-skills`** — the plugin your teammates install. Add skills here.
- **`marketplace-admin`** — owner-only tooling. Don't include in teammate install instructions.
