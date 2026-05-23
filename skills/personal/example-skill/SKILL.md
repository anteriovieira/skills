---
name: example-skill
description: Placeholder skill showing the SKILL.md frontmatter format. Replace with a real skill before promoting to engineering/, productivity/, or misc/. Use when user asks to see the skill template or wants an example of skill structure.
---

# Example Skill

This is a placeholder. Replace its contents with the actual instructions, references, and assets the skill needs.

## Structure

A skill folder typically contains:

- `SKILL.md` — required. Frontmatter (`name`, `description`) plus the body the agent reads when the skill triggers.
- Optional supporting `.md` files — referenced from `SKILL.md` via relative links for progressive disclosure.
- Optional `scripts/` — helper scripts the skill invokes.

## Promoting this skill

1. Move the folder into `skills/engineering/`, `skills/productivity/`, or `skills/misc/`.
2. Add an entry in the top-level `README.md` under the matching section.
3. Add the relative path to `.claude-plugin/plugin.json` `skills[]` array.
4. Update the bucket's `README.md` index.
