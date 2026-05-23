# Antério Vieira's Skills

[![skills.sh](https://skills.sh/b/anteriovieira/skills)](https://skills.sh/anteriovieira/skills)

Personal agent skills I use across Claude Code and other coding agents.

## Install

```bash
npx skills@latest add anteriovieira/skills
```

Pick the skills you want and the agent(s) you want to install them on.

## Layout

```
.claude-plugin/plugin.json      # installable skill manifest
skills/
  engineering/                  # daily code work
  productivity/                 # daily non-code workflow
  misc/                         # kept around but rarely used
  personal/                     # tied to my own setup, not promoted
  in-progress/                  # drafts not yet shipped
  deprecated/                   # no longer used
```

Only skills listed in `.claude-plugin/plugin.json` are exposed to the installer.

## Reference

### Engineering

_None yet._

### Productivity

_None yet._

### Misc

_None yet._

## Scripts

- `scripts/link-skills.sh` — symlink every `SKILL.md` folder into `~/.claude/skills` for local use.
- `scripts/list-skills.sh` — list every `SKILL.md` path in the repo.

## License

[MIT](./LICENSE)
