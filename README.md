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
- A GitHub account with permission to create private repos
- A code editor (e.g. VS Code) to open the cloned repo in

> The `gh` CLI is used under the hood for private repo access. If a step below fails because `gh` isn't installed or authenticated, just ask Claude Code: *"install the GitHub CLI and authenticate with my GitHub account"* — then retry the failing step.

## First-time setup — Claude Code

Click **Use this template** → **Create a new repository** at the top of this page. Set visibility to **Private**, name it (e.g. `acme-skills-marketplace`), and create. From here on, that new repo — not this template — is your marketplace.

### 1. Clone the repo locally

The easiest way is to ask Claude Code to do it for you:

> *clone `<your-username>/<your-new-repo>` into ~/code and open it*

Or run the commands manually:

```
gh repo clone <your-username>/<your-new-repo>
cd <your-new-repo>
code .
```

### 2. Install the management plugin (first time only)

In the **Claude Desktop app**:

- **Customise** → **+** button → **Create Plugin** → **Add marketplace**
- Enter your repo as `<your-username>/<your-new-repo>`
- Install the `marketplace-admin` plugin from the marketplace that appears

Or, equivalently, from a Claude Code session:

```
/plugin marketplace add <your-username>/<your-new-repo>
/plugin install marketplace-admin@team-marketplace
```

> *(The marketplace is called `team-marketplace` at this point — that's the pre-init default. Step 3 renames it.)*
>
> **If install fails:** the `gh` CLI probably isn't installed or signed in. Ask Claude Code to install and authenticate `gh`, then retry.

### 3. Run setup with the management skill

Open the local clone from step 1 in Claude Code. From a session inside that directory, say:

> *setup my marketplace*

The `marketplace-manager` skill walks you through:

- Your company name
- Your marketplace name (replaces `team-marketplace`)
- License (MIT or UNLICENSED)

It rewrites all the placeholders across both runtimes' manifests, swaps this marketing README for a teammate-facing one with install commands inline, and commits.

### 4. Push to GitHub

The skill pushes for you. (If you skipped that prompt, just run `git push`.)

### 5. Refresh the plugin

Your marketplace was renamed in step 3, so update the plugin reference in your runtime:

- **Claude Desktop app:** Customise → marketplace → **Update** (or **Refresh**)
- **CLI:** `/plugin marketplace update <new-marketplace-name>`

### 6. Give teammates access and share the repo

GitHub → your repo → **Settings** → **Collaborators and teams** → invite each teammate's GitHub account (or grant a team read access). Without this, their install will fail with a not-found error.

Then share the repo URL. After step 3, the README on the front page has copy-paste install commands inline for both Claude Code and Codex.

## First-time setup — Codex CLI

Same shape as Claude Code, with one difference at the end: Codex has no auto-update, so you reload your Codex session to pick up the renamed marketplace.

### 1. Clone the repo locally

Same as above — `gh repo clone <your-username>/<your-new-repo>`, `cd` in, open in your editor.

### 2. Install the management plugin

```
codex plugin marketplace add <your-username>/<your-new-repo>
codex plugin install marketplace-admin@team-marketplace
```

If this fails because `gh` isn't set up, install/authenticate it and retry.

### 3. Run setup

From a Codex session inside the local clone:

> *setup my marketplace*

Same prompts as Claude Code (company name, marketplace name, license).

### 4. Push to GitHub

The skill pushes for you.

### 5. Reload Codex

Quit and restart your Codex session, then pull the renamed marketplace:

```
codex plugin marketplace upgrade <new-marketplace-name>
```

### 6. Give teammates access and share the repo

Same as Claude Code step 6.

## Adding a new skill

Once `marketplace-admin` is installed, you can add skills from anywhere on disk — no need to be inside the marketplace clone.

1. Open the repo or directory where the skill lives in Claude Code or Codex.
2. Trigger the management skill: *"add this skill to the team marketplace"* (name the skill if there are several to choose from).
3. The skill copies the skill into your `team-skills` plugin (or whichever plugin you point it at), bumps the plugin version, commits, and pushes.
4. Teammates pick it up on their next runtime startup (with auto-update on) or by running an upgrade command.

> **Recommend to your team:** in Claude Code, run `/plugin`, select your marketplace, and toggle **Enable auto-update**. New skills land automatically on next startup. Codex teammates run `codex plugin marketplace upgrade <marketplace>` periodically.

## Adding a new plugin (e.g. sales, marketing, ops)

When one `team-skills` plugin starts feeling too broad, split into domain plugins. From anywhere on disk, trigger the management skill:

> *add a new plugin called `sales-skills` to the marketplace*

It scaffolds the plugin directory, registers it in both runtimes' manifests, commits, and pushes. Teammates install the new plugin selectively:

```
/plugin install sales-skills@<your-marketplace-name>
# or
codex plugin install sales-skills@<your-marketplace-name>
```

## Other ongoing flows

The management skill also handles:

- *"publish to the marketplace"* — commits and pushes pending changes.
- *"marketplace status"* — shows pending changes and recent ships.

It saves your local clone's path at `~/.config/claude-marketplace-admin/config.json` so it works from any working directory. Both runtimes' manifests stay in sync automatically — never edit one without the other.

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
