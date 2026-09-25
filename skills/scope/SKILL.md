---
name: scope
description: Defines required, optional, and deliberately excluded work before implementation starts, with a one-line reason per exclusion. Use before non-trivial work, especially before delegating to another agent, or when a request is broad enough to read two ways ("while you're in there", "should this include"). Produces a scope statement; implements nothing.
---

# Scope

Say what the work is, what it isn't, and why, before any of it starts.

The failure this prevents is not disagreement about scope. It's scope that was never stated, so nobody could disagree with it. Work expands into adjacent territory because the territory was there, or stops short of something the requester assumed was obvious, and both are discovered at review time when the cost of the misunderstanding is already paid.

This skill acts as a **soft governor** on the others. It doesn't block anything. It states a boundary the rest of the work can be checked against, which is what makes drift visible instead of gradual.

## When this fires

Before implementation on non-trivial work. Most valuable in two situations:

- **The request could be read more than one way.** If two competent people would produce different work from the same sentence, the sentence needs a scope statement before it needs an implementation.
- **The work is being delegated to another agent.** This is where the skill earns its keep. An agent handed a vague task doesn't stop and ask. It fills the gap with a plausible interpretation and starts building. Scope stated up front is the difference between delegation and gambling.

Skip it when the work is small and unambiguous. A scope statement for a one-line fix costs more attention than the ambiguity it resolves, and running it everywhere teaches people to skim past it in the cases that mattered.

## Workflow

### 1. State the requested outcome in the requester's own terms

Use their words. Not a cleaned-up version, not your reading of what they must have meant.

This step looks trivial and is the one that fails. Restating a request in your own vocabulary is already an act of interpretation: "make the upload faster" quietly becomes "optimize the image pipeline," and the substitution is invisible because the new phrasing sounds more precise. It is more precise. That's the problem: precision that wasn't in the original came from somewhere, and that somewhere was you.

If their phrasing is genuinely ambiguous, keep it as-is and name the ambiguity as an open question. An ambiguous request accurately recorded is more useful than a clear request nobody made.

### 2. List the necessary work

What must be done for the stated outcome to be true. Each item should be something whose absence means the outcome wasn't achieved.

Test each entry by removing it: if the outcome still holds without it, it isn't required. It's related, and it belongs in the next list.

### 3. List related work, flagged separately

Things that touch this work, would be reasonable to do, and are not required for the stated outcome. Refactors the change makes tempting. Adjacent bugs noticed on the way. Tests for nearby code. The cleanup someone would do while they're in there.

Separating these is the point. Related work isn't bad work. It's work that needs its own decision rather than arriving as a passenger on this one. Flag each with roughly what it would cost, so the decision to include it can be made rather than defaulted into.

### 4. List what is explicitly not being done, each with a one-line reason

The list that makes this skill worth running.

Adjacent work someone might reasonably expect to be included, stated as excluded, with a reason. Not a justification, not a paragraph. One line. "Not migrating the old callers: they work, and doing both in one change makes the revert unusable."

**A silent exclusion is indistinguishable from an oversight.** That's the whole mechanism. Nobody can tell whether the thing you didn't do was a decision or something you missed, and the reason is what settles it. It also gives the requester something specific to push back on if the reasoning is wrong.

**Don't pad this list.** Twenty things you're not doing is noise, and noise here is expensive because it hides the two exclusions that would have surprised someone. Only list adjacent work a reasonable person might have expected to be included. If nobody would expect it, leaving it out isn't a decision worth recording.

See `references/boundary-patterns.md` for the recurring ways scope silently expands or silently shrinks, and for drawing the required/related line when it's genuinely unclear.

## Scope changes are fine. Silent scope changes are not.

A scope statement is a starting position, not a contract. Work reveals things, and a boundary drawn before the work began will sometimes turn out to be drawn wrong.

When that happens, say so and re-scope. Name what changed, what moved across the line, and why. The rule is not "stick to the original scope"; it's that the boundary is always stated, so moving it is a visible act rather than a gradual one.

A skill that made scope changes feel like failures would get avoided, and then nothing gets scoped at all. Treat a mid-work re-scope as the skill working.

## Output

```
SCOPE: [the request, in the requester's own words]

Required:
  - [work whose absence means the outcome wasn't achieved]

Related, not included:
  - [adjacent work, with rough cost] (needs its own decision)

Not doing:
  - [excluded work]: [one-line reason]

Open questions:
  - [genuine ambiguity in the request, left unresolved rather than assumed away]

Scope:
  [one or two sentences: what this work covers and what it doesn't]
```

`Open questions` stays even when empty. Write "none." An empty section says the request was checked for ambiguity. A missing section says nothing, and reads as though it was.

## Interaction with other skills

- **`verify`** asks how you'll know the work succeeded; this asks what the work is. They chain: the required list here becomes the observable done-criteria in `verify` step 1. Scope defines the boundary, verify proves the inside of it was built.
- **`impact`** traces what a change reaches, which is a different question from what work is in bounds. Run scope first. It's cheaper and often shows the change is smaller than assumed. If `impact` later finds reach nobody expected, that's a reason to re-scope.
- **`reflect`** sometimes produces lessons about scope itself: a category of work that keeps getting silently included, or an exclusion that keeps turning out wrong. Those route well as an addition to the boundary patterns reference.
