# Install: Claude Code

This is the teammate install guide. The owner of this marketplace will give you the exact commands with the real values filled in — but here's the general shape so you know what to expect.

## Prerequisites

- **[Claude Code](https://claude.ai/code)** — CLI, Desktop app, or IDE extension. Any works.
- **[`gh` CLI](https://cli.github.com/)** — installed and authenticated:
  ```
  gh auth login
  gh auth status   # confirm "Logged in to github.com"
  ```
  You need to be signed in as a GitHub user with **read access** to the marketplace repo.

If you use SSH instead of HTTPS for git, your SSH key needs to be loaded in `ssh-agent` and the GitHub host must be in your `known_hosts`. HTTPS via `gh auth` is the simpler path.

## Install (2 commands)

In Claude Code, run:

```
/plugin marketplace add <github-owner>/<repo-name>
/plugin install team-skills@<marketplace-name>
```

Then open the `/plugin` UI → **Marketplaces** tab → find this marketplace → toggle **Enable auto-update**.

That's it. New skills the owner ships will land automatically on your next Claude Code startup.

## Manual update (if auto-update is off)

```
/plugin marketplace update
```

## Troubleshooting

**`/plugin marketplace add` fails with an auth error.** Run `gh auth status`. If you're not signed in, `gh auth login` and pick HTTPS. If you are signed in but it still fails, you may not have read access to the repo — ask the marketplace owner to confirm you're a collaborator on the repo (or member of the org with access).

**`/plugin install` says the plugin doesn't exist.** Run `/plugin marketplace update` first — your local copy of the marketplace may be stale.

**Skills don't trigger when expected.** Each skill has a `description` field that determines when Claude reaches for it. If a skill exists but Claude isn't using it, talk to the owner — that's usually a description tuning problem.

**Headless / CI environments.** Set `GH_TOKEN` (or `GITHUB_TOKEN`) so Claude Code can pull marketplace updates without an interactive `gh` session. The token needs `repo` scope to read the private marketplace.

## Removing the marketplace

```
/plugin uninstall team-skills
/plugin marketplace remove <marketplace-name>
```
