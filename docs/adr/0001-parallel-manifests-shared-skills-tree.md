# Parallel hand-maintained manifests with a single shared skills tree

To support both Claude Code and Codex CLI from the same template, we ship parallel manifest files (`.claude-plugin/marketplace.json` + `.agents/plugins/marketplace.json` at the repo root, and `.claude-plugin/plugin.json` + `.codex-plugin/plugin.json` per plugin) but keep a single `plugins/<name>/skills/` tree shared between both runtimes. The `marketplace-manager` skill is the only writer of these files in normal operation and updates both sets in lockstep on init, add-plugin, and publish.

## Considered options

- **Generate Codex files from Claude source.** Rejected: adds a build step before `codex plugin marketplace add` works, and a fork-owner who forgets to run sync ships a broken Codex install. Cuts directly against the template's "10 minutes, no infra" promise.
- **Invent a template-internal unified manifest and generate both.** Rejected: fights upstream conventions, requires maintainers to learn a third schema, and the per-ecosystem deltas (`policy`, `category`, object `source` on the Codex side) are small and stable enough that the duplication isn't painful.
- **Parallel hand-maintained files (chosen).** Both manifest sets are tracked, both are written by the skill in lockstep. Drift risk is mitigated because the skill is the only writer in normal operation.

The skills tree is shared because `SKILL.md` format is byte-for-byte identical across the two ecosystems (same YAML frontmatter, same description-triggered lazy loading). Splitting it would create duplication with no benefit; sharing it makes "publish once, run anywhere" the default.

## Consequences

- The `marketplace-manager` skill flows must touch two files for every registry-level change and two files for every per-plugin change. This is enforced procedurally in `SKILL.md`, not by tooling.
- Sentinels (`REPLACE_ME_*`) now appear in expanded set of files; init must rewrite all of them. See `CLAUDE.md` for the canonical list.
- Reference: [mvanhorn/last30days-skill](https://github.com/mvanhorn/last30days-skill) ships this same dual-layout pattern in production.
