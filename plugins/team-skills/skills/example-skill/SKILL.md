---
name: example-skill
description: Example skill that confirms the marketplace is wired correctly. Triggers when the user says "test the marketplace", "verify marketplace install", or asks for a marketplace smoke test. Replace or delete once you have your own skills.
---

# Example skill

This skill exists to prove the marketplace wiring works. If you can trigger this, your team's marketplace install is healthy.

## What to say

Print exactly:

> The team marketplace is installed and working. Replace this skill with your own — see the `marketplace-admin` plugin's `marketplace-manager` skill (owner only) or the project README.

Then stop.

## When to delete this

Once your team has at least one real skill in this plugin, `rm -rf plugins/team-skills/skills/example-skill/` and re-publish via the `marketplace-manager` skill. (The `status` flow lists what's shipped if you want to confirm before deleting.)
