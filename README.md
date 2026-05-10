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

## Prerequisites

- [Claude Code](https://claude.ai/code) **or** [Codex CLI](https://github.com/openai/codex) (or both) installed
- [`gh` CLI](https://cli.github.com/) authenticated (`gh auth login`) — used for private repo access and visibility checks
- A code editor (e.g. VS Code) to open the cloned repo in
- A GitHub account with permission to create private repos

## Owner setup walkthrough

There are two things you'll do: get a **personalized copy of this template on GitHub**, and **install the management skill** into your runtime so it can do the rest. The walkthrough below is explicit about which is which — once you've done it, ongoing skill management is a one-line natural-language trigger from anywhere on disk.

### 1. Create your private repo from this template

At the top of this GitHub page, click the green **Use this template** → **Create a new repository** button. On the next screen:

- Set **Visibility** to **Private**.
- Name the repo whatever you want (e.g. `acme-skills-marketplace`).
- Click **Create repository**.

This creates a brand-new repo under your GitHub account with all of this code copied in. From here on, that new repo — *not* this template — is your marketplace. Everything below operates against the new repo.

### 2. Clone the new repo locally and open it in your editor

```
gh repo clone <your-username>/<your-new-repo>
cd <your-new-repo>
code .   # or your editor of choice
```

The local clone is the **working copy**: it's where the management skill will edit files, commit, and push back to GitHub. You'll trigger the skill from a Claude Code or Codex session running inside this directory, so open it in your editor now.

### 3. Confirm `gh` is signed in

```
gh auth status
```

If not signed in: `gh auth login`. The `gh` CLI is what authenticates the marketplace install — for you in the next step, and for each teammate later — so it must be set up first.

### 4. Install the management skill into your runtime

This is a separate track from the local clone: the local clone is the working copy, but the runtime install is how the **management skill itself** gets loaded so you can trigger it. The first command points your runtime at your new GitHub repo; the second pulls the admin plugin from it.

**Claude Code:**
```
/plugin marketplace add <your-username>/<your-new-repo>
/plugin install marketplace-admin@team-marketplace
```

**Codex CLI:**
```
codex plugin marketplace add <your-username>/<your-new-repo>
codex plugin install marketplace-admin@team-marketplace
```

> The marketplace name is `team-marketplace` here because that's the default *before* you've personalized the repo. Step 5 renames it to whatever you choose.

### 5. Run setup by triggering the management skill

From a Claude Code or Codex session **inside the local clone from step 2** (your editor's integrated terminal works), say something like:

> *set up the marketplace*

The `marketplace-manager` skill activates and asks you for:

- Company name
- Marketplace name (replaces `team-marketplace` across both runtimes' manifests)
- License preference (MIT or UNLICENSED)

It then rewrites every `REPLACE_ME_*` placeholder in the manifests and configs, swaps this marketing README for a teammate-facing one with install commands inline, commits, and pushes back to your GitHub repo. Your marketplace is now personalized and ready for teammates.

### 6. Give teammates access to the private repo

The marketplace lives in a private repo, so each teammate's GitHub account needs read access before they can install from it. On GitHub:

- Your repo → **Settings** → **Collaborators and teams** → **Add people** (or **Add teams**)
- Invite each teammate's GitHub account (or grant a team read access — preferred for groups of more than a few)

Without this, the `marketplace add` command on their machine will fail with a not-found / permission error.

### 7. Share the repo URL with teammates

Send them the link to your private repo. After step 5, the README they see on landing is the teammate-facing one with the exact install commands inline for both Claude Code and Codex — no docs/ folder spelunking required.

For reference, the longer install guides are at [`docs/install-claude-code.md`](docs/install-claude-code.md) and [`docs/install-codex.md`](docs/install-codex.md).

### Updates

- **Claude Code:** teammates open `/plugin` → **Marketplaces** tab → toggle **Enable auto-update** once. New skills land on next startup. Manual fallback: `/plugin marketplace update`.
- **Codex CLI:** Codex has no auto-update toggle yet. Teammates run `codex plugin marketplace upgrade <marketplace-name>` to pull updates.

### Adding skills going forward

From any Claude Code or Codex session **anywhere on your machine** (no need to be inside the clone — the skill remembers its path), trigger the management skill with phrases like:

- *"import this skill into the team marketplace"* — copies a skill from `~/.claude/skills/<name>/` or `~/.codex/skills/<name>/` into the marketplace.
- *"publish to the marketplace"* — commits and pushes pending changes.
- *"marketplace status"* — shows pending changes and recent ships.

The skill saves your local clone's path at `~/.config/claude-marketplace-admin/config.json` so it can find the working copy from any cwd. Both runtimes' manifests stay in sync automatically.

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

## About

Built by [Brad Bonanno](https://www.youtube.com/@bradbonanno) at [Solaris Automation](https://www.solarisautomation.io/) — we help teams ship AI-powered workflows without rebuilding infra. Subscribe on [YouTube](https://www.youtube.com/@bradbonanno) for walkthroughs of templates like this one.

## License

MIT in this template (replace during init if you want UNLICENSED for proprietary internal use).
