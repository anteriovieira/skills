# Antério Vieira Skills

Personal collection of agent skills (slash commands and behaviors) loaded by Claude Code and other coding agents via [skills.sh](https://www.skills.sh/).

## Language

**Skill**:
A self-contained unit installed under a coding agent's skills directory. Defined by a `SKILL.md` file with YAML frontmatter (`name`, `description`) and optional supporting files in the same folder.

**Bucket**:
A top-level category folder under `skills/` that groups skills by intent (`engineering`, `productivity`, `misc`, `personal`, `in-progress`, `deprecated`).

**Plugin manifest**:
The `.claude-plugin/plugin.json` file. Lists which skill directories are publicly installable via `npx skills@latest add anteriovieira/skills`. Skills not listed here exist in the repo but are not offered to installers.
