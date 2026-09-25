---
name: adversary
description: Adversarially reviews an already-implemented change across selected attack surfaces, one read-only reviewer per surface, and synthesizes a PASS / PASS WITH CHANGES / FAIL verdict. Reports only, never modifies code. Run manually with /adversary after implementation and before merge, on non-trivial changes. Not for pre-implementation design questions; that is dev-council.
disable-model-invocation: true
---

# Adversary

Attack a change that has already been written, across surfaces chosen for that change, and hand back one synthesized verdict. Finds problems; fixes nothing.

The question here is narrow and worth stating plainly: **did we build it correctly?** That is not the same question as *should we build this*, and it must not be answered by whoever answered that one. A reviewer who approved the design has a stake in the design, and post-hoc review from that position tends to confirm the earlier call rather than test it. Keep the two separate even when it means re-reading context the designer already had.

## When this fires

After implementation, before merge or deploy, on non-trivial changes.

Skip it when the change is trivial. A review that runs on everything gets skimmed on everything, and the finding that mattered gets skimmed with it. The cost of this skill is attention, and attention spent on a config bump is attention unavailable for the migration next week.

**Non-goals.** Doesn't modify code, doesn't apply its own findings, doesn't rewrite the diff. Doesn't evaluate whether the feature should exist. Doesn't replace a pre-implementation design review, and shouldn't be run by the same reviewer that performed one.

## Inputs

- **The diff.** The actual changeset, whether `git diff <base>...HEAD`, a PR, or named files. Not a description of the change.
- **The intent.** One paragraph: what is this change trying to accomplish? Derive it from the commits, the PR body, the request that prompted the work, or the code. If you can't state the intent, ask before reviewing. A reviewer with no intent to measure against invents one, and then reviews the change against something nobody was building.
- **The surfaces to review.** Selected, not defaulted. See below.

Reviewers challenge whether the change *achieves* the intent. They don't challenge whether the intent was right. That boundary is what keeps this from silently becoming a design review.

## Workflow

### 0. Pin the diff before anything else

Resolve the base (`git rev-parse <base>`), confirm the diff against it is non-empty, and record the exact diff command and commit list once. Every reviewer gets that same command. A bad ref or an empty diff should fail here, at the cost of one command, not inside four parallel reviewers that each start cold and each report "nothing to review."

If a scope statement exists for this work (from `scope`, a spec, an issue, or the PR body), locate it now. It decides whether the scope-conformance surface below is available.

### 1. Select the surfaces

Eleven surfaces are available: correctness, scope conformance, security, data integrity, architecture, concurrency, failure handling, performance, maintainability, testing, operations. Full definitions and selection cues are in `references/surface-rubrics.md`.

**Select. Do not run all eleven.** Running every surface on every change is the mechanical version of this skill, and it fails in a specific way: ten reviewers each obligated to produce findings will produce findings, and the real one arrives buried in eight speculative ones. A change with no concurrent access doesn't need a concurrency reviewer, and asking for one anyway teaches the reader to skim.

Three or four surfaces is typical. Choose them from what the change actually touches, and **name the ones you didn't select and why**. That list goes in the output. A surface skipped deliberately is a decision; a surface skipped silently is a gap wearing a decision's clothes.

### 2. One reviewer per surface, independently

Spawn one read-only subagent per selected surface, in parallel. Each gets:

- The stated intent
- The diff
- Its own surface rubric, **only its own**
- The specific context that surface needs (the schema for data integrity, the auth model for security)

Not the whole repository, and not the other surfaces' rubrics. Independence is the point: a reviewer that has read the security rubric will find security issues from the performance seat, and the overlap between surfaces stops being evidence of anything. Correlated reviewers produce agreement that looks like confirmation and isn't.

Where the host allows it, vary the model across reviewers as a secondary source of diversity. Be honest about how much that buys: reviewers from one vendor share priors and blind spots, so within a single family this is a weaker signal than genuine cross-vendor diversity would be. Surface separation is doing the real work here; model variation is a bonus, not the mechanism.

**Keep the fan-out cheap.** This step is where the skill's cost lives: every reviewer starts cold.

- Hand each reviewer the diff and file paths, not pasted file contents. It opens only what its surface needs.
- Where the host lets you choose a subagent's model (in Claude Code, the Agent tool's `model` parameter), run reviewers on a mid-tier model such as `sonnet`. Keep the strongest model for the synthesis in step 3, where the judgment is. A surface that needs the stronger model (subtle concurrency, a security boundary) can have it; that is a per-change call, not a default.
- Cap each reviewer's report at about 400 words: findings with evidence, nothing restated from the diff or the intent. A reviewer with more to say than that is usually padding weak findings around one strong one, and the cap forces it to rank.

### 3. Synthesize

You are the lead reviewer with full context, not a neutral aggregator that concatenates subagent output. Reviewers each saw a slice. You know the constraints, the timeline, and which tradeoffs were already made and rejected. Use that.

Sort every finding:

- **Critical.** Breaks correctness, security, or data integrity. Blocks merge.
- **High-confidence.** Raised independently by two or more surfaces, or raised once with evidence strong enough to stand alone. Independent arrival at the same problem from different angles is the strongest signal available here.
- **Single-surface.** One reviewer, real but unconfirmed. Report it as such; don't inflate it to consensus and don't drop it.
- **Disagreements.** One reviewer flags what another explicitly clears. Do not resolve these by picking the more confident one. Say what each saw, and what would settle it.
- **Ruled out.** Raised and rejected, each with a one-line reason.

**Show the ruled-out findings.** A synthesis that only reports what survived is asking to be trusted rather than checked, and it hides the judgment that did the most work. Listing what was dismissed and why lets the reader overturn a call you got wrong, which is the entire reason a separate reviewer exists.

### 4. Verdict

- **PASS.** No critical findings, nothing high-confidence that should block. Ship it.
- **PASS WITH CHANGES.** Specific findings to address, none of which require rethinking the approach. Name them explicitly; a "pass with changes" that doesn't say which changes is a pass.
- **FAIL.** A critical finding, or enough high-confidence findings that the approach itself is in question.

The verdict follows the findings and the reasoning, not a count. Six maintainability nits is a PASS. One unverified permission boundary is not. If the verdict is genuinely close, say which way and what would decide it rather than rounding to the cleaner-looking answer.

## Output

```
ADVERSARY: [the change reviewed, in one line]

Intent:
  [one paragraph, what this change was trying to do]

Surfaces reviewed: [the selected ones]

Findings:
  Critical:
    - [finding, surface, evidence, why it blocks]
  High-confidence:
    - [finding, which surfaces raised it independently, evidence]
  Single-surface:
    - [finding, surface, why it's unconfirmed]
  Disagreements:
    - [what each reviewer saw, and what would settle it]
  Ruled out:
    - [finding, who raised it, one-line reason for rejecting it]

Not checked / not applicable:
  - [surfaces deliberately not selected, each with why]
  - [anything a reviewer couldn't reach: no credentials, no test data, external service]

Verdict:
  [PASS | PASS WITH CHANGES | FAIL]
  Reasoning: [what drove it, naming the specific finding that set it]
  Required before merge: [only for PASS WITH CHANGES / FAIL, the specific changes]
```

## Interaction with other skills

- **`impact`** runs before the change and maps what it will reach. This runs after and attacks what was written. If `impact` flagged consumers, they are a strong candidate for the correctness or data-integrity surface here.
- **`scope`** produces the statement the scope-conformance surface measures against. Without one, that surface is skipped and the output says so.
- **`verify`** asks whether the change was proven to work at runtime. This asks whether it is correct in ways runtime evidence wouldn't surface. Complementary: a change can pass its evidence block and still fail here.
- **A pre-implementation design review** asks whether to build it. Deliberately a different reviewer, for the anchoring reason at the top of this file.
