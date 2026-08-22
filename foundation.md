# Zoth Skills - Foundation

For anyone, human or AI agent, adding a new skill to this repo, read this first. This is the substrate every skill sits on. None of the rules here were invented for this document: every one of them already appears, restated in slightly different words, in at least three of the individual skill specs. This document exists so they get written once and inherited, not repeated and allowed to drift out of sync with each other.

If a new skill's spec seems to need a rule not covered here, that's a signal to check whether it belongs here instead of in the individual spec.

---

## The four rules every skill inherits

### 1. Report, don't auto-fix, for anything in a review/investigate role

`impact`, `adversary`, `investigate`, `distill`, `clarity` all analyze and propose. None of them modify code or prose unilaterally. This isn't caution for its own sake; it's a role boundary: these skills exist to surface things the implementing agent (or Zoth) then decides on. A skill that both finds the problem and silently fixes it removes the decision point that's the entire reason to run a separate check in the first place.

### 2. No absolute/mechanical rules; judgment, shown

This came directly out of building `clarity` against pstack's actual "avoid em dashes entirely" rule, but it's not specific to that skill. It applies anywhere a skill is tempted to say "always do X" or "never do Y" instead of "here's the pattern, here's why it matters in this instance." Frequency and context decide, not category. Every flag or finding a skill produces should show its reasoning, not just assert a verdict.

### 3. Evidence over assertion, and say what wasn't checked

`verify`'s evidence block ("Not verified: R2 production bucket visibility, flagged for manual check") is the clearest version of this, but the underlying rule is general: a skill's output should never let an unchecked thing pass silently as if it were checked. If a skill couldn't verify something, that goes in the output as its own line, not an omission.

### 4. Named-incident bar before a skill gets built

Every skill spec in the roadmap was tested against "can you name a real incident this targets, not a hypothetical." Several failed that test and got deferred. This isn't a one-time filter applied when the roadmap was written; it's a standing bar. Before treating any deferred skill as ready to build, check whether a real incident has actually shown up yet. If not, it's still premature, roadmap entry or not.

**Exception, used twice now, worth stating as a rule rather than re-litigating each time:** a skill can proceed without a named incident if it has a genuine *structural* argument instead, not "this sounds useful" but a real sequencing or architectural reason the skill needs to exist before evidence of its necessity can even be collected. `adversary` was the first case: pre- and post-implementation review are different questions, so the case for building it doesn't rest on a specific past failure. `scope` is the second: it's meant to govern every other skill, so there's a real reason to have it exist before the skills it governs are all built, rather than after. `distill` was checked against this exception and doesn't qualify: no structural argument, just "this would probably help," so it stays deferred on the plain rule, with the additional reason that it's meant to review against what `verify`'s incident log has caught, and that log is still empty.

The test for whether something qualifies as a structural exception rather than a rationalization: can you state the sequencing or architectural reason in one sentence without using the word "useful" or "helpful"? If the justification is about value rather than necessity or order, it's not a structural exception, it's the plain rule applying.

### 5. Public core, private instance data, required because these ship to other people

These skills are going to public, global install: anyone can add `verify` or `impact` to their own setup. That means the *mechanism* each skill implements is shared and shipped, but anything that's actually Zoth's own data is not:

- `verify`'s incident log ships **empty**, as a template, not pre-filled with Zoth's own past incidents. Each installer's log fills in from their own project's history.
- `impact` doesn't hardcode ZothKit's package map or the phased-extraction-plan doc as fixed context; it takes *a* project's dependency structure as input, generically. The R2 migration stays in the skill's docs as an illustrative example of the problem class, not as baked-in context the skill assumes exists.
- `clarity`'s calibration section learns *the installer's* voice, not "Zoth's voice" specifically; the mechanism (flag a pattern more than once, ask whether it's actually the user's style) is generic, and whose voice it calibrates to is per-installation.
- Named incidents used throughout the roadmap (R2 migration, the QA-background test bar) stay as motivating examples in documentation, real and useful for explaining *why* a skill exists, but never as data the shipped skill depends on at runtime.

Rule of thumb: if removing a piece of content would make the skill stop working for someone who isn't Zoth, it's private instance data and doesn't belong in the shipped skill. If it would just make the documentation less concrete, it's a fine example to keep.

---

## Shared output shape

Skills in a review/report role (impact, adversary, investigate, distill, clarity) should converge on a recognizable output shape so a human reading any of them doesn't have to relearn the format each time:

```
[SKILL NAME]: [what was analyzed]

Findings:
  - [specific finding, with reasoning, not just a verdict]
  - [specific finding, with reasoning]

Not checked / not applicable:
  - [anything the skill couldn't verify or deliberately skipped, stated explicitly]

Verdict / recommendation:
  - [the actual output: a risk rating, a pass/fail, a proposed change list,
     whatever is specific to that skill, but always present and always last]
```

Individual skills specify what goes in "Findings" and what "Verdict" means for them (a risk rating for `impact`, a PASS/FAIL for `adversary`, a removal list for `distill`). The shape, findings, then explicit gaps, then verdict, stays constant so the skills read as one system, not ten unrelated tools.

## Interface decision: skill vs. subagent

Every skill spec should declare one of these, and the choice should follow this logic rather than be arbitrary:

- **Subagent** (isolated context, returns a summary) when the skill does independent investigation that would otherwise pollute the main conversation with intermediate noise: tracing dependents, running parallel adversarial reviews, exploring an unfamiliar code path. `impact`, `adversary`, `distill` fit this.
- **Skill** (inline, loaded into the main conversation) when the value is in the discipline being visible and followed in real time, not delegated away: defining scope before work starts, requiring an evidence block at the end of a task, reviewing prose you're about to publish. `scope`, `verify`, `clarity` fit this.
- `investigate` is the one genuinely ambiguous case in the roadmap, noted there as a live decision, not resolved here, because it depends on whether Zoth wants to watch the hypothesis elimination happen or have it handed back as a finished report.

## Minimum-context discipline (absorbs what `context` was reaching for)

`context` was deferred as its own skill because the source material admits it isn't really invokable on its own; it's a discipline the subagent-based skills should already practice. Stated as a rule instead of a tenth skill: any subagent-based skill (`impact`, `adversary`, `distill`) should be given the task, the specifically relevant files/contracts, and the specifically relevant constraints, not the whole repository by default. If, once these are running, that discipline turns out to need more structure than "remember to scope the context," that's the signal to revisit `context` as an actual skill later, per the roadmap note.

## Naming and directory convention

**Prefix rule, reversed from the earlier decision: no per-skill prefix.** pstack itself, the source this whole roadmap is built from, doesn't prefix its own skills: `architect`, `blast-radius`, `interrogate`, `unslop`, no `pstack-` on any of them. The repo/plugin name carries the identity; individual skill names stay clean. Matching that: skill folders are just `<name>/SKILL.md`, `verify`, `clarity`, `impact`, `adversary`, and so on, with no `zoth-` prefix baked in. The repo is already named `zoth-skills`; repeating that inside every folder name states the namespace twice.

The collision-risk concern that motivated the earlier prefix decision is real, and checking it against current Anthropic issue tracker data makes it firmer than it was when this decision was first made: plugin skills currently have **no automatic namespacing at all**, confirmed via `anthropics/claude-code` issue `#20994` ("Skill auto-namespacing does not work as documented") and `#50486` (an open feature request to bring skill namespacing up to parity with commands, which already get it: `/plugin-name:command`). Skills currently use exactly the raw YAML `name:` field, unprefixed, with no plugin-scoped disambiguation. This is not a reliability bug that sometimes fires wrong; it's a feature that hasn't shipped for skills yet.

Practical consequence for this repo, stated plainly: `verify`, `scope`, `impact`, `clarity`, `reflect`, `adversary` currently have zero platform-level collision protection. If another installed plugin defines a skill with the same name, plausible, since these are short, generic, guessable names, there is currently no mechanism distinguishing yours from theirs. This was accepted knowingly when the prefix was dropped, on the understanding that packaging as a real plugin would eventually close the gap once Anthropic ships `#50486`. It hasn't shipped as of this check. Revisit this decision if that risk becomes uncomfortable rather than treating "packaged as a plugin" as if it already solved it.

- Skill folders: `<name>/SKILL.md`, with `references/` for anything the skill needs loaded on demand (pattern lists, incident logs, playbooks) rather than inlined in the main SKILL.md body
- Every SKILL.md frontmatter `description` should state both what it does and when it triggers, pushily enough that it doesn't undertrigger. This is a general Claude-skill authoring convention, not specific to zoth-skills, but worth holding to consistently across all of them.
- **Resolved:** codify (encoding a verification-failure lesson into a test/type/invariant/CI check) is a step inside `verify`'s incident-log workflow, not its own skill. `reflect` stays a real, separate skill, but narrower than first drafted: scoped to lessons from finished work where nothing failed a check (a design insight from a successful migration, not a bug). If a lesson came from something that passed checks and still broke, it belongs in `verify`'s log. If it came from work that succeeded but taught something worth keeping, it belongs in `reflect`.

---

## What this replaces in the individual skill specs

Going forward, individual skill specs in the roadmap don't need to restate "shows reasoning," "doesn't silently rewrite," or "reports rather than fixes"; those are inherited from here. Each skill's spec should only describe what's actually specific to it: its goal, its named incident (or lack of one), its particular workflow steps, and what "Findings" and "Verdict" mean in its specific output. If a skill's spec starts repeating a rule that's already in this document word for word, that's a sign that section can be deleted and replaced with "see foundation."