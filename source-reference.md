# Zoth Skills - Source References

Read this alongside `foundation.md` and `roadmap.md`. This file exists because everything in the roadmap so far describes pstack's skills secondhand: my summaries of them, verified against search results but never handed to you as direct paths. Before building any zoth skill, your agent should read the actual source file(s) listed below, not just the roadmap's description of them.

## Getting the source

`cursor/plugins` (the canonical repo) blocks direct page fetches via robots.txt. The reliable path is a full clone, which also pulls in nested files a single-page fetch would miss (playbooks, references/ subfolders):

```bash
git clone https://github.com/cursor/plugins.git
# skills live under: plugins/pstack/skills/
```

If that's unavailable, `backnotprop/pstack` is a mirror of the same repo and its individual files were directly fetchable during this conversation:
```bash
git clone https://github.com/backnotprop/pstack.git
# skills live under: pstack/skills/
```

`backnotprop` describes itself as a mirror of `cursor/plugins/pstack`, but the two are **not** identical. A comparison on 2026-09-24 found 52 files under `skills/` differing, with `cursor/plugins` updated more recently (2026-09-23, against the mirror's 2026-09-14). The differences are real wording changes, not formatting: `blast-radius/SKILL.md`, for example, reads differently between the two. Treat `cursor/plugins` as the source of truth and use the mirror only when the canonical clone fails, noting in the build which copy was read.

## Also worth reading: the Claude Code port

`drrius/ronin` is an unofficial port of pstack adapted specifically for Claude Code, Codex, and other Agent Skills hosts (43 curated skills including the 21 principle skills, renamed `ronin-*`, using native Claude Code subagents instead of Cursor's Task API):

```bash
git clone https://github.com/drrius/ronin.git
```

Unofficial and unendorsed by Lauren Tan or Cursor. Read it as a second data point on how someone else solved the Cursor-to-Claude Code adaptation problem, not as verified-correct. Worth checking specifically for how it re-expressed subagent orchestration in Claude Code's format, since that's the hardest part of any of these ports.

## Source path per zoth skill

| Zoth skill | Read from pstack | Notes |
|---|---|---|
| `impact` | `pstack/skills/blast-radius/SKILL.md` | |
| `verify` | `pstack/skills/create-verification-skill/SKILL.md`, `pstack/skills/maintain-verification-skill/SKILL.md`, `pstack/skills/principle-prove-it-works/SKILL.md` | Three separate files; verify covers both defining and maintaining verification surfaces |
| `adversary` | `pstack/skills/interrogate/SKILL.md` | |
| `investigate` | `pstack/skills/poteto-mode/playbooks/bug-fix.md` | **Not** `figure-it-out`, and not `investigation.md` either. Confirmed by direct read: `investigation.md` covers read-only explanation requests with no hypothesis tracking in it. `bug-fix.md` is the actual source for the observe/reproduce/trace/hypothesize workflow this skill needs. |
| `scope` | `pstack/skills/poteto-mode/SKILL.md` | The routing/scope-discipline logic lives in the main poteto-mode file itself |
| `distill` | `pstack/skills/unslop/SKILL.md`, `pstack/skills/principle-subtract-before-you-add/SKILL.md` | Distill is unslop's subtraction philosophy applied to code instead of prose; read both |
| `clarity` | `pstack/skills/unslop/SKILL.md` | Same source as distill, different application: this is the prose-facing half |
| `codify` | `pstack/skills/principle-encode-lessons-in-structure/SKILL.md` | Resolve the overlap with verify's incident log (see roadmap) before building either |
| `reflect` | `pstack/skills/reflect/SKILL.md`, `pstack/skills/recall/SKILL.md` | Same resolve-before-building flag as codify |
| `context` | `pstack/skills/principle-guard-the-context-window/SKILL.md`, `pstack/skills/how/SKILL.md`, `pstack/skills/poteto-mode/SKILL.md` (Subagents section) | The principle is the core; `how` shows scoped explorer briefs; poteto-mode's subagent defaults supply pointers-not-payloads and model per role. `interrogate` and `architect` show the same scoping pattern |

## Second source: mattpocock/skills

`mattpocock/skills` by Matt Pocock (MIT) is a separate collection, not a pstack derivative. It clones directly:

```bash
git clone https://github.com/mattpocock/skills.git
# skills live under: skills/<category>/<name>/
```

| Zoth skill | Read from mattpocock/skills | What was taken, and what changed |
|---|---|---|
| `handoff` | `skills/productivity/handoff/SKILL.md`, `skills/in-progress/claude-handoff/SKILL.md` | Taken: point to artifacts instead of copying them, redact, tailor to the next session's focus, suggest skills. Added: verified vs. assumed state, decisions with rejected options, dead ends, a "verify on pickup" section, one next action, a size budget, and destination rules for ephemeral environments where a temp file won't survive. |
| `adversary` | `skills/engineering/code-review/SKILL.md` | Taken: validate the diff ref once before spawning reviewers; review conformance to the spec as a separate axis; cap reviewer output. Changed: conformance became a selectable surface inside adversary's synthesis rather than a second report that is never merged. |
| all skills (pruning pass) | `skills/productivity/writing-for-agents/SKILL.md`, "Pruning" section | Taken: the no-op test (does this sentence change behavior versus the model's default?) and single-source-of-truth for each meaning. Applied as an editing pass over existing skill bodies; nothing from it ships as its own skill. |
| `investigate` (not built) | `skills/engineering/diagnosing-bugs/SKILL.md` | Second reference beside pstack's `bug-fix.md`: builds a fast failing feedback loop before hypothesizing. Read both before building. |

Same caution as for pstack applies: take the mechanism, check each specific rule against `foundation.md`, don't copy text.

## A caution worth repeating here

Reading the source is for understanding the mechanism and pattern, not for copying text into the zoth version. The whole point of this exercise, from the very first pass on `clarity`, was building on pstack's ideas without inheriting their specific choices uncritically. The em-dash rule is the standing example of a real mistake in the source worth catching, not repeating. Read the original, then check its specific rules against `foundation.md`'s "no absolute/mechanical rules" principle before anything from it goes into a zoth skill.