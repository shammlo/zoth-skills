# Routing Guide — reflect

The seven destinations a lesson can go to, ordered roughly from weakest enforcement to strongest, plus `nothing` at the end as its own category.

**Enforcement strength is the tiebreaker, not the goal.** When two destinations both genuinely fit, take the stronger one — it's the one that doesn't depend on someone remembering. When only a weaker one fits, take it honestly rather than promoting a judgment call into a mechanism that will be wrong in cases nobody anticipated. A suppressed lint rule teaches people to suppress lint rules.

---

## temporary

*A note for the current stretch of work, expected to stop mattering.*

Context that's useful for the next few days: a branch that has to merge before another, a workaround in place until an upstream fix lands, a value that's hardcoded pending a decision.

The distinguishing feature is a known expiry. If you can't say roughly when it stops being true, it isn't temporary — it's a project rule you're avoiding committing to.

Write the expected expiry into the note itself. A temporary note with no expiry becomes permanent by default, and stale context is worse than none because it's read with the same trust as current context.

## documentation

*An explanation someone will need in order to understand the system.*

How a subsystem works, why a non-obvious design is shaped the way it is, what the tradeoffs were. Its job is comprehension.

Route here when the lesson answers "why is this like this" — the question whose answer is otherwise lost with the person who made the decision. This is the natural home for a "we assumed X, it turned out to be Y" finding that doesn't imply a rule.

Docs are read when someone is already looking. That makes this a poor destination for anything that needs to change behavior at the moment of the mistake, and a good one for anything that needs to be understood before the work starts.

## project rule

*A standing instruction: do it this way here.*

A convention specific to this codebase, in whatever file the project's agents and contributors actually read.

The honest question: will anyone read it in time? Project rules work when there are few of them. Every addition dilutes the whole file, and the twentieth rule doesn't get followed — it gets skimmed past on the way to the one someone came for.

Before routing here, check whether a test or a check could carry the same lesson. If so, that's the better home. Route here when the lesson genuinely requires judgment at the moment of application, which a mechanism can't supply.

## test

*A case that fails if the lesson is ever violated.*

Usually the right answer when the lesson is about behavior. A test states the lesson in a form that requires nobody to have read anything — it just fails.

Strong specifically because it's checked automatically, at the moment of violation, by machinery already running. A lesson worth a paragraph of documentation is usually worth a test instead, and the test is the version that still works when nobody reads the paragraph.

Route here when you can name the concrete condition that would signal the lesson was forgotten. If you can't state the failing case, it's probably not a test — it may be an architecture constraint.

## architecture constraint

*A structural rule about what may depend on what.*

Module boundaries, dependency direction, which layer may import which, what is allowed to reach the database. Route here when the lesson is about shape rather than behavior.

Strongest when actually enforced — a dependency-check in CI, a module boundary the build rejects, a type that makes the wrong dependency unrepresentable. A constraint that lives only in prose is a project rule wearing a stronger name, so if it can't be enforced, route it to project rule and be honest that it depends on people remembering.

The strongest rung available anywhere is a state that cannot be represented at all: the wrong thing doesn't fail a check, it fails to compile. Reach for it when the type system can carry the lesson.

## skill

*A change to how an agent behaves on future tasks.*

A new check inside an existing skill, a line in a checklist or rubric, a description that needs to trigger more reliably, or — rarely — a new skill.

**The highest-leverage and least reversible destination.** A skill edit changes behavior on every future task, including tasks nobody anticipated, and a bad edit is hard to notice because it manifests as work quietly done differently.

Two guards:

- **Prefer editing an existing skill over creating a new one.** A lesson about a category of breakage nobody checked for usually belongs as a line in `impact`'s surface checklist or `adversary`'s rubrics, not as a new skill. New skills should clear the same bar as any other: name the real incident it targets.
- **Explicit individual approval.** Skill-route items are listed separately in the output and approved one at a time, never waved through as a batch.

## nothing

*Understood, deliberately not encoded.*

The right answer more often than a reflection habit makes it feel. Route here when the lesson was specific to one change, when a competent person would already know it, when it's already written somewhere findable, or when the cost of another rule exceeds what the rule buys.

**Say it explicitly rather than omitting the item.** "Considered, routed to nothing, because X" is a decision that can be revisited. Silence is indistinguishable from having missed it.

A run of this skill whose honest output is entirely `nothing` is a successful run. The alternative — manufacturing a destination for every observation — produces a rules file nobody reads, and buries the three rules that mattered under thirty that didn't.

---

## Anti-patterns

**Acknowledging without recording.** "Worth keeping in mind" persists nothing. Either it gets a destination or it gets routed to `nothing` on purpose.

**Recording without routing.** A note saying a lint rule should exist is not the lint rule. The route isn't complete until the destination is named and the work to get it there is either done or filed.

**Fixing without generalizing.** Fixing the instance while leaving the pattern intact means the same lesson arrives again next quarter wearing different details.

**Papering over a structural problem with a written rule.** If the real fix is structural, the written instruction is a symptom of not having made it. Route to the mechanism, not to prose about the mechanism.
