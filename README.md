# zoth-skills

Agent skills I build for my own engineering work, published in case they're useful to anyone else.

These follow the [Agent Skills](https://code.claude.com/docs/en/skills) open standard: each skill is a directory containing a `SKILL.md`. Currently tested only on Claude Code.

## Skills

| Skill | What it does |
| --- | --- |
| [dev-council](skills/dev-council) | Runs a hard architecture decision past five advisors with different concerns, peer-reviews their arguments anonymously, and produces a verdict with explicit reversal conditions. Refuses to convene when the decision doesn't warrant it. |

## Install

Copy the skill directory into your skills folder:

```bash
git clone https://github.com/<you>/zoth-skills.git
cp -r zoth-skills/skills/dev-council ~/.claude/skills/
```

Use `~/.claude/skills/` for personal skills or `.claude/skills/` inside a project for project-scoped ones. Each skill has its own README with usage and design notes.

## About

Built by Shamlo, who goes by Zoth. Portfolio and writing: [shamlo.dev](https://shamlo.dev).

Some skills borrow names from a speculative fiction universe I'm writing. The naming is an identity layer; the engineering roles underneath are always stated explicitly.

## License

MIT