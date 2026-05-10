# Install: Codex CLI

This is the teammate install guide. The owner of this marketplace will give you the exact commands with the real values filled in — but here's the general shape so you know what to expect.

## Prerequisites

- **[Codex CLI](https://github.com/openai/codex)** installed.
- **[`gh` CLI](https://cli.github.com/)** — installed and authenticated:
  ```
  gh auth login
  gh auth status   # confirm "Logged in to github.com"
  ```
  You need to be signed in as a GitHub user with **read access** to the marketplace repo.

If you use SSH instead of HTTPS for git, your SSH key needs to be loaded in `ssh-agent` and the GitHub host must be in your `known_hosts`. HTTPS via `gh auth` is the simpler path.

## Install (2 commands)

```
codex plugin marketplace add <github-owner>/<repo-name>
codex plugin install team-skills@<marketplace-name>
```

That's it. The `team-skills` plugin is now installed and its skills are available in any Codex session.

## Updates

To pull the latest skills from the marketplace:

```
codex plugin marketplace upgrade <marketplace-name>
```

Run this whenever the owner ships new skills (or set up a shell alias / cron if you want to automate it).

## Troubleshooting

**`codex plugin marketplace add` fails with an auth error.** Run `gh auth status`. If you're not signed in, `gh auth login` and pick HTTPS. If you are signed in but it still fails, you may not have read access to the repo — ask the marketplace owner to confirm you're a collaborator on the repo (or member of the org with access).

**`codex plugin install` says the plugin doesn't exist.** Run `codex plugin marketplace upgrade` first — your local copy of the marketplace may be stale.

**Skills don't trigger when expected.** Each skill has a `description` field that determines when Codex reaches for it. If a skill exists but Codex isn't using it, talk to the owner — that's usually a description tuning problem.

**Headless / CI environments.** Set `GH_TOKEN` (or `GITHUB_TOKEN`) so Codex can pull marketplace updates without an interactive `gh` session. The token needs `repo` scope to read the private marketplace.

## Removing the marketplace

```
codex plugin uninstall team-skills
codex plugin marketplace remove <marketplace-name>
```
