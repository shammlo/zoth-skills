# Dev Council

A Claude Code skill for engineering decisions that are expensive to get wrong. Five independent advisors analyze the decision, peer-review each other anonymously, and a chairman synthesizes a verdict with explicit reversal conditions.

**The Six:** five engineering advisors and one chairman.

Adapted from [Karpathy's LLM Council](https://github.com/karpathy/llm-council) methodology, retargeted for full-stack development.

## The Six

| Seat | Engineering role | Primary question |
| --- | --- | --- |
| **Nicron** | Chairman | What does the evidence actually support? |
| **Luminarian** | Maintainer | Will this remain sane? |
| **Mammoon** | Attacker | How can this fail or be abused? |
| **Wiseone** | Pragmatist | Do we actually need this? |
| **Azoth** | Scale Engineer | Where does this stop working? |
| **Nareth** | Newcomer | Can someone understand and use it correctly? |

The tensions are the point. Luminarian and Wiseone disagree about whether to build it right or build it now. Mammoon and Wiseone disagree about safe versus fast. Azoth and Nareth disagree about optimal versus legible. Deliberately, no seat argues for *more* architecture.

## What makes it different from asking once

Most multi-agent prompts produce five variations of the same answer. Three mechanisms prevent that here:

**A guard that refuses to convene.** Before anything runs, the decision is scored on reversibility, blast radius, migration cost, consumer count, and compatibility surface, then classified:

- **Level 0**: canonical, local, or cheaply reversible. No council. Direct answer.
- **Level 1**: real tradeoff, bounded blast radius. Three advisors, no peer review.
- **Level 2**: cross-cutting, hard to reverse, or costly to get wrong. All five, plus peer review.

Cost of *being wrong* is scored separately from cost of *reversing*. JWT versus server-side sessions is trivial to swap in week one and still Level 2, because the damage accrues before you notice it.

```
Decision
   │
   ▼
 Guard
   ├── Level 0 → Direct answer
   ├── Level 1 → 3 advisors ─────────────────────────┐
   └── Level 2 → 5 advisors → Anonymous peer review ─┤
                                                     ▼
                                                  Chairman
                                                     │
                                                     ▼
                                      Verdict + confidence
                                      + reversal conditions
```

**Evidence discipline.** Every significant claim is labeled `observed`, `inferred`, `assumed`, or `speculative`. Advisors may not invent codebase behavior, consumers, or performance characteristics, and generic best practices don't count as evidence about your codebase. An advisor asserting "this will become a bottleneck" with nothing behind it gets marked speculative, which makes it easy to discount.

**Evidence over headcount.** Five advisors agreeing doesn't make something correct. One advisor demonstrating that a load-bearing assumption is false can destroy the recommendation regardless of the other four. Peer review is anonymized (A-E) so reviewers weigh arguments rather than deferring to whichever seat "owns" the topic.

## Output

Every verdict ends with the same structure:

```
Where the Council Agrees
Where the Council Clashes
Blind Spots the Council Caught
The Recommendation      (including what NOT to do)
Why
Confidence              (derived from evidence quality, not asserted)
Reversal Conditions     (concrete triggers to revisit this)
The One Thing to Do First
```

"Don't decide yet, keep the current implementation until X" is a first-class outcome, not a failure to reach one.

## Install

Copy this skill directory into your skills folder:

```bash
git clone https://github.com/shammlo/zoth-skills.git
cp -r zoth-skills/skills/dev-council ~/.claude/skills/
```

Use `~/.claude/skills/` for personal use, or `.claude/skills/` inside a project to scope it there.

## Use

Say `council this`, `dev council`, or just ask a real architecture question:

```
council this: should the storage abstraction own presigned URL generation,
or should each app request them through it?
```

```
should I extract this auth middleware into a shared package, or keep it local?
```

Ask it something small and it will tell you it doesn't need a council, then answer. That's working as intended.

## Design notes

The skill is deliberately hostile to architecture for its own sake. Its stated objective is the *simplest* architecture that stays correct under actual constraints, not the most sophisticated one. Consumer evidence beats architectural elegance: "this could be useful later" is not evidence an abstraction is valid. Zero consumers doesn't prove an abstraction is wrong, but it does mean it hasn't earned its complexity yet.

There's a hard budget on codebase scanning (three files, five ceiling) and a no-repetition rule on advisor output, because the failure mode of a council isn't being wrong, it's being expensive and agreeable.

## Known limitations

Stated plainly, since the skill itself demands that assumptions be labeled rather than asserted:

- **Token consumption is not well characterized.** A Level 2 session is eleven inference passes plus a codebase scan. There are guards against runaway cost (a hard three-file scan budget, a 100-200 word cap on advisor output, a no-repetition rule, three reviewers instead of five), but no measured baseline for what a session actually costs versus reasoning through the same decision directly. If you run it, that number would be genuinely useful to know.
- **The tiering thresholds are judgment calls, not calibrated ones.** Level 1 versus Level 2 depends on the model's own assessment of blast radius and cost of being wrong. It may over-classify toward Level 2 on decisions the user is visibly invested in.
- **Anonymization is only as good as the writing.** Advisors are instructed to write in neutral register so their seat isn't identifiable by voice. Whether that holds in practice is unverified.
- **Evidence discipline depends on compliance.** The `observed` / `inferred` / `assumed` / `speculative` labels are instructions, not enforcement. A model can label something `observed` that it inferred.

## Roadmap

- Measure real token cost per level against a direct-reasoning baseline
- Support for other coding agents. The methodology is agent-agnostic, but the file format, tool invocation, and parallel execution are Claude Code specific and would need adapting
- Calibrate tier thresholds against real usage rather than estimates

## About the names

The council seats are named for entities from a speculative fiction universe I'm writing. Built by Shamlo, who goes by Zoth. If you're curious: [shamlo.dev](https://shamlo.dev).

## License

MIT