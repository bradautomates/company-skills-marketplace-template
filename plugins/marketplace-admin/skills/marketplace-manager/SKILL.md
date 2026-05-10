---
name: marketplace-manager
description: Owner-only tooling for managing a private skills marketplace that serves both Claude Code and Codex CLI from the same repo. Use when the user says "set up the marketplace", "init the marketplace", "add a skill to the team marketplace", "import this skill into the marketplace", "publish to the marketplace", "ship it", "add a plugin to the marketplace", "check marketplace status", "what's in the marketplace", "list skills", "what have I shipped", "rename the marketplace", or otherwise wants to administer their team's shared skills repo. Handles first-run init, importing existing skills from anywhere on disk (~/.claude/skills, ~/.codex/skills, project dirs), scaffolding new plugins, publishing (commit + push), and reporting status. Writes parallel manifests for both Claude Code (.claude-plugin/) and Codex (.agents/, .codex-plugin/) in lockstep. Not for authoring skills from scratch — use the built-in skill-creator / write-a-skill skill for that, then import the result here.
---

# Marketplace manager

You administer a private skills marketplace repository on behalf of its owner. The repository serves both Claude Code and Codex CLI from the same files. Five flows: `init`, `import`, `add-plugin`, `publish`, `status`. Pick the flow that matches the user's intent, then follow its procedure.

## Dual-runtime layout

Every change touches **two manifest sets** in lockstep:

| Layer | Claude Code path | Codex path |
|---|---|---|
| Marketplace registry (repo root) | `.claude-plugin/marketplace.json` | `.agents/plugins/marketplace.json` |
| Plugin manifest (per plugin) | `plugins/<name>/.claude-plugin/plugin.json` | `plugins/<name>/.codex-plugin/plugin.json` |
| Skills tree (per plugin) | `plugins/<name>/skills/` (shared — both runtimes read it) | same |

Whenever a flow writes to one manifest, it must also write the parallel file. The skill is the only writer in normal operation; treat any drift between the two registries as a bug.

## Conventions you must enforce

- **Names are kebab-case.** Marketplace name, plugin names, skill names. Auto-fix obvious typos (`escalationTriage` → `escalation-triage`, `Escalation Triage` → `escalation-triage`) and confirm with the user before writing.
- **No name collisions.** Before creating a plugin or importing a skill, list existing entries and reject duplicates.
- **Manifests are the source of truth.** Read them every flow — never rely on cached state.
- **Sentinel strings are `REPLACE_ME_*`.** Their presence anywhere in the repo means init has not been completed for that field.
- **Codex defaults.** When scaffolding new entries in `.agents/plugins/marketplace.json`, default `policy.installation: "AVAILABLE"`, `policy.authentication: "ON_INSTALL"`, `category: "Productivity"`. Do not add an `interface` block to plugin.json on init — the owner can add display metadata later if they ever publish externally.

## Locating the marketplace repo

Owners often invoke this skill from a directory unrelated to the marketplace repo. To find the repo:

1. Read `~/.config/claude-marketplace-admin/config.json` if it exists. Expected shape: `{"repo_path": "/abs/path/to/marketplace-repo"}`.
2. If the file is missing or `repo_path` doesn't exist on disk:
   - If the current working directory contains `.claude-plugin/marketplace.json` (or `.agents/plugins/marketplace.json`), use cwd and offer to save it to config.
   - Otherwise ask the user for the path, verify both registry files exist at that path, then save it.
3. Always operate against the resolved `repo_path` — never assume cwd.

When writing the config file, create `~/.config/claude-marketplace-admin/` if it doesn't exist. Use `mkdir -p`.

## Flow: `init` (first run)

Trigger when: any `REPLACE_ME_*` sentinel exists in any manifest, `README.instance.md`, or `LICENSE`.

Procedure:

1. **Confirm cwd is the marketplace repo.** If the user invoked from inside the cloned template, cwd should contain both `.claude-plugin/marketplace.json` and `.agents/plugins/marketplace.json` with sentinels present. If not, ask them to `cd` into the cloned template repo and re-trigger.
2. **Verify it's a git repo with a GitHub remote.** Run `git remote get-url origin` to capture `<owner>/<repo>`. If no remote, prompt the user to either (a) create one with `gh repo create <name> --private --source=. --remote=origin --push`, or (b) provide the slug manually.
3. **Confirm privacy.** Run `gh repo view --json visibility -q .visibility`. If not `PRIVATE`, prompt: "This repo is currently public. Make it private now? (recommended — your team's skills shouldn't be world-readable.)" If yes: `gh repo edit --visibility private --accept-visibility-change-consequences`.
4. **Collect:**
   - Marketplace name (kebab-case, default suggestion derived from repo name)
   - Owner display name (the company / team name — appears in plugin UIs)
   - License preference: `MIT` (keep), `UNLICENSED` (proprietary, recommended for private internal repos), or `other` (prompt for SPDX identifier)

   Do **not** ask about Codex-specific metadata (display name, category, capabilities, brand color, etc.). Defaults are sufficient for private team marketplaces; the owner can edit `.codex-plugin/plugin.json` later if they ever publish externally.
5. **Rewrite files in lockstep across both runtimes:**
   - `.claude-plugin/marketplace.json`: replace `team-marketplace` with new name (only if user changed it), replace `REPLACE_ME_company_name` everywhere
   - `.agents/plugins/marketplace.json`: replace `team-marketplace` with new name (only if user changed it), replace `REPLACE_ME_company_name` in the `interface.displayName` field
   - `plugins/team-skills/.claude-plugin/plugin.json`: replace `REPLACE_ME_company_name`
   - `plugins/team-skills/.codex-plugin/plugin.json`: replace `REPLACE_ME_company_name`
   - `plugins/marketplace-admin/.claude-plugin/plugin.json`: no sentinels by design (description is generic)
   - `plugins/marketplace-admin/.codex-plugin/plugin.json`: no sentinels by design
   - `README.instance.md`: replace `REPLACE_ME_marketplace_name`, `REPLACE_ME_company_name`, `REPLACE_ME_github_owner/REPLACE_ME_github_repo` with real values
   - `LICENSE`: replace `REPLACE_ME_company_name` on line 3. If user chose UNLICENSED, replace the file body with an UNLICENSED stub; if they chose another SPDX identifier, replace with the corresponding license text.

   Validate that no `REPLACE_ME_*` sentinels remain anywhere in the repo before continuing. If any do, surface them and stop.
6. **Swap READMEs:** confirm the destructive rename with the user, then `git rm README.md && git mv README.instance.md README.md`. The marketing README is unrecoverable post-init by design (different audience, dates badly).
7. **Save config:** Write `~/.config/claude-marketplace-admin/config.json` with `{"repo_path": "<absolute path of repo>"}`.
8. **Commit and push:**
   ```
   git add -A
   git commit -m "chore: initialize marketplace as <name> for <company>"
   git push -u origin <branch>
   ```
9. **Print teammate install snippets for both runtimes** (with real `<owner>/<repo>` and `<marketplace-name>` substituted):
   ```
   # Claude Code
   /plugin marketplace add <owner>/<repo>
   /plugin install team-skills@<marketplace-name>

   # Codex CLI
   codex plugin marketplace add <owner>/<repo>
   codex plugin install team-skills@<marketplace-name>
   ```
   Remind the owner to share `docs/install-claude-code.md` and `docs/install-codex.md` with their team — teammates pick whichever runtime they use.
10. **Print owner reminder:** "From now on, trigger this skill from anywhere — your repo path is saved at `~/.config/claude-marketplace-admin/config.json`."

## Flow: `import` (add an existing skill to the marketplace)

Trigger when: user says "add this skill", "import skill", "publish skill X to team", "share my skill", etc. Init must already be complete (no sentinels remain).

Procedure:

1. **Identify the source skill.** Ask if not given: "What's the path to the skill you want to import?" Common locations:
   - `~/.claude/skills/<name>/` (Claude Code user-level personal skills)
   - `~/.codex/skills/<name>/` (Codex user-level personal skills)
   - `<some-project>/.claude/skills/<name>/` or `<some-project>/.agents/skills/<name>/` (project-level skills)
   - Any directory with a `SKILL.md` at its root.
   Verify the path contains `SKILL.md`. Read the frontmatter to get the skill `name`. (Skills are byte-for-byte identical across runtimes, so the source can come from either ecosystem.)
2. **Pick the target plugin.** List plugins from `<repo>/.claude-plugin/marketplace.json` (excluding `marketplace-admin`). If exactly one, default to it and confirm. If multiple, ask which.
3. **Check for collisions.** If `<repo>/plugins/<plugin>/skills/<skill-name>/` already exists, ask: overwrite, rename, or abort.
4. **Copy.** Use `cp -R <source> <repo>/plugins/<plugin>/skills/<skill-name>`. Preserve the entire directory (sub-files, scripts, references — anything the SKILL.md depends on).
5. **Verify kebab-case** on the skill directory name; rename if needed.
6. **Offer cleanup of the source copy.** Ask: "Delete the original at `<source>`? You'll consume the team version once you install `<plugin>@<marketplace>` on this machine, which avoids divergence between your personal copy and the team copy." If yes: `rm -rf <source>`. Tell the user to run the install command in whichever runtime they use (`/plugin install` for Claude Code, `codex plugin install` for Codex) if they haven't already.
7. **Hand off to publish.** Print: "Ready to publish? I can commit and push now, or you can keep editing first." If yes, run the `publish` flow.

## Flow: `add-plugin` (scaffold a new plugin in the marketplace)

Trigger when: user says "add a plugin", "split into plugins", "create a sales plugin", etc.

Procedure:

1. **Collect plugin name** (kebab-case) and one-line description.
2. **Reject collisions** with existing plugin names in either `.claude-plugin/marketplace.json` or `.agents/plugins/marketplace.json`.
3. **Create the directory structure (both manifest dirs):**
   ```
   plugins/<name>/
     .claude-plugin/plugin.json
     .codex-plugin/plugin.json
     skills/                  (empty directory; create with a .gitkeep)
   ```
4. **Write `.claude-plugin/plugin.json`:**
   ```json
   {"name": "<name>", "version": "0.1.0", "description": "<description>"}
   ```
5. **Write `.codex-plugin/plugin.json`:**
   ```json
   {
     "name": "<name>",
     "version": "0.1.0",
     "description": "<description>",
     "license": "<license-from-LICENSE-or-MIT>",
     "skills": "./skills/"
   }
   ```
6. **Append to `.claude-plugin/marketplace.json`** — add `{"name": "<name>", "source": "./plugins/<name>", "description": "<description>"}` to the `plugins` array.
7. **Append to `.agents/plugins/marketplace.json`** — add to the `plugins` array:
   ```json
   {
     "name": "<name>",
     "source": {"source": "local", "path": "./plugins/<name>"},
     "policy": {"installation": "AVAILABLE", "authentication": "ON_INSTALL"},
     "category": "Productivity"
   }
   ```
8. **Validate both registry files parse as JSON** (use `jq . <file>` or equivalent) before continuing.
9. **Hand off to publish.**

## Flow: `publish` (commit + push)

Trigger when: user says "publish", "ship it", "push", "release", or after import/add-plugin completes.

Procedure:

1. Run `git -C <repo_path> status --short`. If no changes, print "Nothing to publish" and stop.
2. Show the user the changed files (skills added/modified, plugins added).
3. **Bump versions where appropriate:** for each plugin whose `skills/` or own files changed, bump the patch number in **both** `plugins/<plugin>/.claude-plugin/plugin.json` and `plugins/<plugin>/.codex-plugin/plugin.json` to the same value. Versions must stay in sync across the two files. Skip if user prefers to bump manually.
4. **Compose commit message** in the form `<verb>: <skill-or-plugin-name> in <plugin-name>` for single-skill changes, or a summary for multi-file changes. Verbs: `add`, `update`, `remove`.
5. **Commit and push:**
   ```
   git -C <repo_path> add -A
   git -C <repo_path> commit -m "<message>"
   git -C <repo_path> push
   ```
6. Confirm: "Claude Code teammates with auto-update enabled will pick this up on their next startup; manual fallback is `/plugin marketplace update`. Codex teammates need to run `codex plugin marketplace upgrade <marketplace-name>` (Codex has no auto-update toggle as of writing)."

## Flow: `status`

Trigger when: user says "what's in the marketplace", "list skills", "marketplace status", "what have I shipped".

Procedure:

1. Read both `<repo>/.claude-plugin/marketplace.json` and `<repo>/.agents/plugins/marketplace.json`. List each plugin (name, version from its plugin.json, description).
2. **Drift check.** Compare the two registries: the same plugin set should appear in both, with identical `name` values. If they diverge, warn loudly — that's a bug. Likewise, for each plugin compare `version` between `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json` and warn if they differ.
3. For each plugin, list skills under `plugins/<plugin>/skills/<*>/SKILL.md` with their `name` and `description` from frontmatter.
4. Run `git -C <repo_path> status --short` and `git -C <repo_path> log -5 --oneline` to show pending changes and recent ships.
5. If any `REPLACE_ME_*` sentinels are still present anywhere, warn the user and suggest re-running the `init` flow.

## What you do NOT do

- **You do not author skills.** If the user asks you to write a new skill from scratch, redirect them to the built-in `write-a-skill` / `skill-creator` skill. Once they have a working skill, return here for the `import` flow.
- **You do not manage teammate installs.** Teammate-side installation lives in `docs/install-claude-code.md` and `docs/install-codex.md`. If the owner asks how to onboard a teammate, point at the doc that matches the teammate's runtime.
- **You do not let the two registries drift.** Every flow that writes one registry must write the other. Treat any divergence between `.claude-plugin/marketplace.json` and `.agents/plugins/marketplace.json` as a bug, and the same for `.claude-plugin/plugin.json` vs `.codex-plugin/plugin.json` within a plugin.

## Reference: file shapes

`.claude-plugin/marketplace.json` (Claude Code):
```json
{
  "name": "<kebab-case>",
  "owner": { "name": "<display name>" },
  "description": "<one line>",
  "plugins": [
    { "name": "<kebab>", "source": "./plugins/<kebab>", "description": "<one line>" }
  ]
}
```

`.agents/plugins/marketplace.json` (Codex):
```json
{
  "name": "<kebab-case>",
  "interface": { "displayName": "<display name>" },
  "plugins": [
    {
      "name": "<kebab>",
      "source": { "source": "local", "path": "./plugins/<kebab>" },
      "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
      "category": "Productivity"
    }
  ]
}
```

`.claude-plugin/plugin.json`:
```json
{
  "name": "<kebab-case>",
  "version": "<semver>",
  "description": "<one line>"
}
```

`.codex-plugin/plugin.json`:
```json
{
  "name": "<kebab-case>",
  "version": "<semver>",
  "description": "<one line>",
  "license": "<spdx>",
  "skills": "./skills/"
}
```

A skill is `<plugin>/skills/<skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`) plus body. The same `SKILL.md` is read by both runtimes; bodies lazy-load on description match in both. Skills may include sub-files, scripts, and other resources in the same directory.
