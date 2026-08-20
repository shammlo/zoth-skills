# zoth-skills

Agent skills I build for my own engineering work, published in case they're useful to anyone else.

These follow the [Agent Skills](https://code.claude.com/docs/en/skills) open standard: each skill is a directory containing a `SKILL.md`. Currently tested only on Claude Code.

## Skills

Listed roughly in the order you'd reach for them across a piece of work.

| Skill | What it does |
| --- | --- |
| [scope](skills/scope) | Defines what work is required, what's related but optional, and what's deliberately not being done, before implementation starts. Every exclusion carries a one-line reason, so a boundary is a decision rather than an oversight. |
| [dev-council](skills/dev-council) | Runs a hard architecture decision past five advisors with different concerns, peer-reviews their arguments anonymously, and produces a verdict with explicit reversal conditions. Refuses to convene when the decision doesn't warrant it. |
| [impact](skills/impact) | Maps what a change reaches before it's made: direct and indirect dependents across code, data, security, infrastructure, jobs, tests, and backwards compatibility, ending in a risk rating with its reasoning shown. Separates what it checked and cleared from what it couldn't check at all. |
| [verify](skills/verify) | Requires runtime evidence before a task is called done, rather than "typecheck passes, tests pass, ship it." Closes with an evidence block that states what wasn't verified instead of omitting it. |
| [adversary](skills/adversary) | Attacks an already-implemented change across selected attack surfaces, one independent reviewer per surface, and synthesizes a single verdict. Shows the findings it ruled out, not only the ones that survived. |
| [reflect](skills/reflect) | Captures lessons from work that succeeded and routes each one to where it should actually live: a test, a rule, an architecture constraint, a skill, or nowhere. Treats "nowhere" as a real answer rather than a failed run. |
| [clarity](skills/clarity) | Reviews prose for AI-generated tells before publishing, sorted into three tiers by how much context decides whether the pattern is a problem. Flags and proposes; never rewrites silently, and never bans a construction outright. |

They're built to work as one system rather than as separate tools. Each one reports and proposes instead of changing things on its own, shows the reasoning behind a finding instead of asserting a verdict, and states what it didn't check rather than letting an unchecked thing pass as a checked one. Skills that need a longer pattern list, checklist, or log keep it in a `references/` directory and load it on demand.

## Install

Copy the whole set into your skills folder:

```bash
git clone https://github.com/shammlo/zoth-skills.git
cp -r zoth-skills/skills/* ~/.claude/skills/
```

Or take a single skill:

```bash
cp -r zoth-skills/skills/impact ~/.claude/skills/
```

Use `~/.claude/skills/` for personal skills or `.claude/skills/` inside a project for project-scoped ones. Skills that ship a log or calibration list start empty and fill in from your own projects, so nothing arrives pre-loaded with someone else's history.

## About

Built by Shamlo, who goes by Zoth. Portfolio and writing: [shamlo.dev](https://shamlo.dev).

Some skills borrow names from a speculative fiction universe I'm writing. The naming is an identity layer; the engineering roles underneath are always stated explicitly.

## License

MIT
