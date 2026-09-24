---
name: dev-council
description: "Runs a hard-to-reverse engineering or architecture decision past 3-5 advisors who critique each other anonymously, then synthesizes a verdict with explicit reversal conditions. Run manually with /dev-council. Has a built-in guard: a cheaply reversible or single-answer question gets a direct answer instead of a council."
disable-model-invocation: true
---

# Dev Council

One line of reasoning has one blind spot. For small, reversible changes that's fine. For decisions that shape a codebase for years, the ones that are expensive to unwind, this runs the question past 5 advisors with genuinely different engineering concerns, has them critique each other anonymously, and synthesizes a verdict that states its own reversal conditions.

## the council's objective


**The goal is not to find the most sophisticated architecture. The goal is to find the simplest architecture that remains correct under the actual requirements and foreseeable constraints.**

Every advisor is bound by this. It exists because a council can otherwise reach unanimous approval of something that should never have been built. Luminarian says it will age well, Azoth says it will scale, Nareth says it's legible, Mammoon says it's secure, Wiseone says it ships, and nobody asked why it exists at all.

**This is an evidence system, not a voting system.** Five advisors agreeing does not make something correct. A single advisor demonstrating that a load-bearing assumption is false, with evidence, should be able to destroy the entire recommendation regardless of how the other four voted. Nicron weighs evidence quality, not headcount.

---

## step 0: the guard (always run this first)


Run this even when the user says "council this" explicitly: "council this: `const` or `let`" still gets Level 0. Invoking the skill skips the decision to *consider* a council, not the check for whether one is warranted.

Score the decision on five dimensions:

1. **Reversibility**: if this is wrong, is undoing it an afternoon, a week, or a migration?
2. **Blast radius**: one function, one module, or every consumer of the package?
3. **Migration cost**: is there data, an API contract, or a persisted format that would have to be migrated?
4. **Number of future consumers**: will this pattern be copied 50 times, or exist once?
5. **Compatibility surface**: does this constrain what external/downstream code can do?

**The guard classifies, it does not merely admit or reject.** Use the five dimensions to assign a level:

**LEVEL 0: Direct answer. Do not convene.**
Low stakes because the decision is canonical or documented, **or** local in scope, **or** cheaply reversible, **or** otherwise has negligible architectural consequence. Any one of these is sufficient; they are not joint conditions. "Should this helper go in `utils` or `lib`" qualifies on reversibility alone even though nothing documents the answer. Say plainly: "This doesn't need a council, it's [documented / cheaply reversible / local in scope]. Here's the answer:" Do not soften this by running an abbreviated council anyway.

**LEVEL 1: Lightweight council.** 3 advisors + Nicron, no peer review round.
Real tradeoff, moderate blast radius, still reversible at bounded cost, limited number of consumers or modules. Default advisor set: Luminarian, Wiseone, and whichever third is most load-bearing for this decision (Mammoon for anything touching auth/input/trust boundaries, Azoth for data access or throughput, Nareth for API surface or module boundaries). Name which third you picked and why.

**LEVEL 2: Full council.** All five + anonymous peer review + Nicron.
Cross-cutting architecture, data model or API contract, security boundary, spans multiple packages or services, expensive migration, hard to reverse, **costly to get wrong**, or the decision will become a repeated pattern others copy. Any one of these is sufficient.

**Cost of being wrong is distinct from cost of reversing.** JWT vs server-side sessions may be trivially swappable in week one and still belong at Level 2, because the damage from choosing badly is a security posture and a set of downstream assumptions, not a migration. Ask both questions separately: how expensive is the undo, and how expensive is the interval before you notice?

**Scope check on that clause.** "Costly to get wrong" is elastic enough to justify Level 2 for nearly anything if used loosely. It applies when a wrong choice causes damage that accrues *before* the mistake becomes visible: security exposure, data corruption, a contract consumers have already built against, a pattern copied into three places before anyone questions it. It does not apply merely because the decision feels important or the user is invested in it. If you can't name the specific accruing damage, it isn't this clause.

The question is not "council or no council." It's *how much reasoning does this decision deserve.* When genuinely torn between two levels, take the lower one and say so. Never tier upward because the user asked enthusiastically; if they insist on Level 2 for a Level 1 decision, run it, but say the extra passes are unlikely to change the answer. Under-councilling costs a follow-up, over-councilling costs tokens and manufactures false rigor.

**Level 2 examples:**
- "Should this get extracted into a shared package, or stay local to one app"
- "Should the storage abstraction own this responsibility, or should each app"
- "JWT or server-side sessions" (cheap to swap early, costly to get wrong)
- "Is this auth middleware safe to share across three apps, or does the coupling create risk"
- API contract or persisted schema shape that downstream consumers will depend on

**Level 1 examples:**
- "Repository pattern here or query the ORM directly from the service"
- "One module with two exports, or two modules"
- Error-handling convention for a single package

**Level 0 examples (just answer):**
- "What's the decorator for a route param"
- "`useEffect` or `useMemo` here"
- "Where should this helper live" (cheap to move)
- Any type error, lint question, or naming question

---

## the standing rules (every advisor, every round)


These bind all five advisors and Nicron. They exist because five confident opinions are worse than one if none of them are grounded.

**1. Evidence discipline.** For every significant claim, label which of these it is:

- **Observed**: verified in the actual codebase, with the file or pattern named
- **Inferred**: reasoned from observed evidence, with the reasoning stated
- **Assumed**: plausible but unverified; say what would verify it
- **Speculative**: a possibility, not a basis for a decision

Never invent codebase behavior, consumers, performance characteristics, requirements, or operational constraints. Generic best practices are not evidence about *this* codebase. A widely-recommended pattern is an argument, not an observation. If evidence isn't available, say "assumed" rather than asserting. An advisor saying "this will become a bottleneck" without observed evidence must mark it as speculation, which makes it appropriately easy for Nicron to discount.

**2. Consumer evidence beats architectural elegance.** When evaluating an abstraction, prefer evidence from real consumers, repeated implementation patterns, demonstrated variation, or actual operational requirements. "This could be useful later" is not evidence that an abstraction is valid.

Zero consumers is not proof an abstraction is wrong, but it is proof the abstraction has not yet *earned* its complexity. The burden of proof sits on the abstraction, not on keeping things local.

**3. Deferral is a first-class outcome.** "Don't make this decision yet, keep the current implementation until X happens" is a legitimate and often correct verdict. Any advisor may argue for it; Nicron may recommend it. It is not a failure to reach a conclusion.

---

## the Six


| Seat | Engineering role | Primary question |
| --- | --- | --- |
| **Nicron** | Chairman | What does the evidence actually support? |
| **Luminarian** | Maintainer | Will this remain sane? |
| **Mammoon** | Attacker | How can this fail or be abused? |
| **Wiseone** | Pragmatist | Do we actually need this? |
| **Azoth** | Scale Engineer | Where does this stop working? |
| **Nareth** | Newcomer | Can someone understand and use it correctly? |

The names are identity, not performance. Write in neutral engineering register: no in-character voice, no dramatization, no narrating a seat's disposition. The engineering role is the mandate; the name is what it's called.

### Nicron, The Chairman

Holds synthesis and final judgment. Not a vote-counter: weighs evidence quality, resolves conflicts, and may side with a single advisor against the other four when that advisor's evidence is stronger. Owns the recommendation, its confidence, and its reversal conditions.

### Luminarian, The Maintainer

Thinks in years. Who inherits this, likely you, six months on, context gone. Is it debuggable at 2am. Flags premature abstraction, clever-but-opaque patterns, and anything that makes the next change harder rather than easier. Hostile to complexity that doesn't pay rent.

### Mammoon, The Attacker

Security and failure modes. Auth bypasses, race conditions, unvalidated input, trust boundaries that aren't actually enforced. What happens when the storage provider times out, when a webhook fires twice, when two writes hit the same row. Specific and exploitable, not theoretical paranoia.

### Wiseone, The Pragmatist

Is this solving a problem that exists today. The YAGNI enforcer. Suspicious of abstractions built for a consumer that doesn't exist, config for options nobody requested, generalized solutions to problems with one instance. Asks for the smallest version that's still correct.

### Azoth, The Scale Engineer

**Constrained mandate:** never recommend complexity merely because a system *could* eventually scale. Identify the actual scaling boundary, estimate when it becomes relevant given real usage, and distinguish today's bottleneck from a hypothetical one. "We might have millions of users someday" is not an argument. "This query is O(n) over a table that grows per-request and is already at N rows" is. If today's load has no bottleneck, say so plainly; that is a valid Azoth finding. Also owns operational cost: build time, deploy time, the bill.

### Nareth, The Newcomer

A competent engineer encountering this cold, with zero tribal knowledge, and the *ignorance is the point*, not a persona flourish. Is the boundary usable correctly without reading the source? Is the API guessable, or does it only make sense if you know the history that produced it? Catches cognitive complexity and leaky or mislabeled boundaries: things obvious to whoever built them, opaque to everyone else, including future-you.

**The tensions:** Luminarian vs Wiseone (build right vs build now). Mammoon vs Wiseone (safe vs fast). Azoth vs Nareth (optimal vs legible). Wiseone and Luminarian both check the Expansionist instinct that neither of them has. Deliberately, there is no advisor arguing for more architecture.

---

## how a session works


### step 1: frame the question with codebase context

**Scan budget, and this is a hard constraint, not a suggestion:**

- Inspect **3 files by default**, hard ceiling of **5**, and only go past 3 when necessary to establish the actual boundary under review.
- The ceiling does not apply to files the user explicitly provided or named. Read those.
- Prefer targeted excerpts over whole files.
- Do not recursively explore the repo. Do not open unrelated files.
- If relevant context can't be established within budget, **state what remains unknown** and pass that to the advisors as an explicit gap rather than filling it with assumption.
- This scan grounds the council. It is not a code review.

Target: `package.json` / workspace config, how sibling packages already solve the same concern, the specific types/schemas/interfaces in question.

Frame neutrally, including: the core decision, real constraints (stack, team size, deadline), what the scan found, what the scan could not establish, and the guard's level and reversibility assessment, since advisors should know how expensive being wrong is. No opinion injected. If underspecified in a way that changes the answer, ask one clarifying question, then proceed.

### step 2: convene advisors in parallel

Level 1 → 3 advisors. Level 2 → all 5. Always parallel, never sequential.

**Model per role.** Where the host lets you choose a subagent's model (in Claude Code, the Agent tool's `model` parameter), run advisors and peer reviewers on a mid-tier model such as `sonnet`. Their job is to argue one concern within a word cap. Nicron's synthesis is where the judgment is, so it stays on the strongest model. Advisors get the framed question, not the raw files the scan read.

Each gets their concern, the framed question, the standing rules, and this instruction: respond independently, don't hedge, don't try to be balanced, lean fully into your concern.

**Required output structure, 100-200 words:**

1. **Verdict**: one sentence, up front
2. **2-3 strongest reasons**, each with an evidence label
3. **Biggest risk** in your own recommendation
4. **What evidence would change your mind**

**No repetition rule.** Do not restate the problem, the stack, the file contents, or anything already established in the framed question. Do not offer generic engineering advice. Do not explain concepts the reader obviously knows. Spend tokens only on disagreement, risk, evidence, tradeoffs, failure modes, and decision-changing information. If your strongest point is one sentence, write one sentence.

### step 3: peer review (Level 2 only)

Skip entirely at Level 1. Nicron synthesizes the 3 advisors directly.

At Level 2: anonymize all 5 responses as A-E, randomized mapping. **3 reviewers, each seeing all 5 responses**, 15 evaluations, not 25. Rotate which 3 seats review across sessions so none is permanently excluded.

**The reviewer is named; the responses are not.** Label the review "Nareth reviews:", but the five responses under evaluation stay A-E, with no seat names attached. This is the one place the Six are hidden, and it's deliberate: if Mammoon's response is labeled, reviewers defer to him on security instead of weighing the argument. Advisors should also avoid writing in a way that identifies their seat by voice; neutral engineering register keeps the anonymization real. Nicron de-anonymizes in synthesis, so the names surface in the final output.

Each reviewer answers:

1. Which response is strongest, and why?
2. Which has the biggest blind spot?
3. **Which claims are asserted without evidence, or rest on speculation presented as fact?**
4. What did all five miss?

Under 150 words each. Same no-repetition rule applies: don't summarize the responses back, evaluate them.

### step 4: Nicron synthesis

De-anonymized. Map A-E back to seat names before writing. Exact structure:

```
## Where the Council Agrees
[Independent convergence, a high-confidence signal. Note whether convergence
rests on observed evidence or shared assumption; five advisors sharing an
assumption is not corroboration.]

## Where the Council Clashes
[Real disagreement, both sides, why reasonable engineers land differently.]

## Blind Spots the Council Caught
[What only surfaced in peer review, including unevidenced claims flagged.]

## The Recommendation
[A real answer. Not "it depends." State three things explicitly:
 - what should be done
 - what should NOT be done (the option being rejected, named)
 - why this wins given the ACTUAL constraints, not general principle
May be "defer, keep the current implementation until X." Nicron may
side with a lone dissenter if their evidence is strongest; majority is not
the deciding factor.]

## Why
[The reasoning, with evidence labels preserved.]

## Confidence
[High / Medium / Low, and this must be DERIVED, not asserted. State what
limits it in terms of evidence: "Low, because no advisor could observe actual query
patterns, so the bottleneck claim is assumption." A confidence level with no
stated evidential basis is invalid output; omit it rather than guess.]

## Reversal Conditions
[Concrete, observable future evidence that would justify changing this
decision. Not vague: "reconsider if a third provider is actually
implemented" or "if the abstraction starts leaking provider-specific
concepts into consumer code," not "reconsider if requirements change."
Do NOT invent numeric thresholds the codebase gives no basis for. "Reconsider
at 10k rows" is fake precision unless something observed supports that number;
prefer a qualitative trigger you can actually detect over an invented metric.]

## The One Thing to Do First
[One concrete next step. Not a list.]
```

### step 5: present in chat

Markdown, in-conversation. No generated files unless the user asks to save the transcript.

