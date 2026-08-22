---
name: reflect
description: "Captures durable lessons from work that succeeded — what happened, which assumptions turned out wrong, what's reusable — and routes each one to where it should actually live. Proposes routings; never applies them unasked. Use after finishing significant work: a shipped feature, a completed migration, a refactor that landed. Trigger on \"reflect\", \"what did we learn\", \"post-mortem\", \"retro\", or finishing a non-trivial piece of work. Not for failures — if something passed its checks and broke anyway, that belongs in `verify`'s incident log instead."
---

# Reflect

Work that succeeds still teaches things, and those lessons have no natural home. A failure leaves a bug report, a broken test, an incident. A success leaves nothing but the working code and whatever the person who wrote it happens to remember next month.

The problem class this targets: an abstraction built during a migration that worked — nothing failed, every check passed, the thing shipped — and only afterward does it become clear the abstraction was shaped around the wrong axis. There was no failure to log. The insight arrived with nothing to attach itself to, and got lost.

## The boundary — read this before anything else

This skill and `verify`'s incident log split the same territory along one line:

- **Something passed its checks and broke anyway** → `verify`'s incident log. That's a verification failure, and its log exists to make future runs check for it.
- **Work succeeded and taught something worth keeping** → here.

The split is by what happened, not by how significant the lesson feels. A lesson from a near-miss that got caught late still belongs in the incident log. A profound realization from a flawless migration belongs here. When a piece of work produced both, split the entry and send each half where it goes rather than filing both in whichever skill you invoked.

## When this fires

After finishing significant work — not on every task. Invoked deliberately, never automatically.

Reasonable triggers: a migration completed, a feature shipped that took real design work, a refactor that changed how something is structured, an approach that was corrected partway through and the corrected version generalizes.

**Skip when the work was routine.** A skill that runs after every task produces a lesson after every task, and lessons that get produced on schedule are not lessons — they're a log with ambition. The bar is that something was actually learned that a competent person on this codebase wouldn't already know.

## The bar: one-offs are not learnings

Before anything gets routed, it has to clear this. Ask of each candidate:

- Will this recur, or was it specific to this one change?
- Would someone competent already know it, or is it genuinely particular to this codebase?
- Is it already written down somewhere that a person would actually find?

A thing that fails all three is a memory, not a lesson, and the honest route for it is **nothing**. Which is a real outcome — see below.

## Workflow

### 1. What happened

Two or three sentences. What was the work, what approach was taken, what did it turn out to be. Written for someone with no memory of it, because in six months that's who's reading.

### 2. Which assumptions turned out wrong

The highest-value part, and the part most often skipped because successful work makes its own assumptions look retroactively correct. Something was believed at the start that turned out not to hold — the work succeeded anyway, which is exactly why it's easy to miss.

Be specific about what was believed and what was actually true. "The storage layer was assumed to be the varying part; the access pattern turned out to be" is a lesson. "Some assumptions were revisited" is not.

### 3. What's reusable

What would you tell someone starting similar work tomorrow? Name the transferable part, not the specific fix. A fix applies once; the shape of the problem applies repeatedly.

### 4. Route each lesson

Seven destinations. Full definitions and selection guidance in `references/routing-guide.md`:

**temporary** · **documentation** · **project rule** · **test** · **architecture constraint** · **skill** · **nothing**

Two rules govern the choice:

**Prefer the strongest rung that fits.** A lesson enforced by something that fails loudly beats one enforced by something a reader has to notice and remember. Roughly strongest to weakest: a state made unrepresentable, then a check that fails CI, then a test, then a documented rule, then a note. If a lesson could be a test or a paragraph, it's a test — the paragraph requires everyone to read it, and the test requires nobody to.

Don't force it. The ladder ranks mechanisms by strength, not by preference, and a judgment call encoded as a lint rule becomes a rule people learn to suppress. If it genuinely requires judgment, write it down prominently with an example of the failure mode, and don't pretend a mechanism can carry it.

**"Nothing" is a first-class outcome.** Say it plainly and move on when it's right.

A reflection skill that always produces a rule will produce rules that shouldn't exist, and every one of them dilutes the ones that should. Nobody reads the twentieth project rule. If a session's honest output is "we learned something, it doesn't need to be encoded anywhere," that's a successful run of this skill, not a failed one.

### 5. Propose — don't apply

Present the routing and stop. Nothing gets written to a rules file, a skill, or a test suite until whoever asked has seen the full list and approved it.

This matters most for the **skill** route. A skill edit changes behavior for every future task, which makes it the highest-leverage and least reversible destination on the list. It gets explicit approval, individually, not as part of a batch.

Show what was dropped, with reasons. The candidates that didn't clear the bar are part of the output — they show where the line was drawn, and let someone overturn a call that was wrong.

## Output

```
REFLECT — [the work being reflected on]

What happened:
  [2-3 sentences]

Assumptions that turned out wrong:
  - [what was believed] → [what was actually true]

Reusable:
  - [the transferable shape, not the specific fix]

Not kept:
  - [candidate considered and dropped, and which bar it failed]

Routing:
  - [lesson] → [destination] — [why this rung and not a stronger or weaker one]
  - [lesson] → nothing — [why it doesn't need a home]

  Belongs in `verify`'s incident log instead:
  - [any lesson that came from something breaking despite passing checks]

  Needs approval before applying: [the skill-route items, listed separately]
```

## Interaction with other skills

- **`verify`'s incident log** owns lessons from things that broke despite passing checks, including deciding whether each becomes a test, type, invariant, or CI check. This skill covers the other half of the same territory, and the two should never hold the same entry.
- **`impact`** and **`adversary`** are candidates for the *skill* route: a lesson about a category of breakage nobody looked for is often best encoded as a line in `impact`'s surface checklist or `adversary`'s rubrics, rather than as a new rule somewhere nobody reads.
