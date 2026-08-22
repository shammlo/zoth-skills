# Zoth Skills - Build Roadmap (v2)

**Read `foundation.md` and `source-reference.md` before this file, and before building anything.** `foundation.md` defines the shared rules, output shape, and interface-decision logic the specs below assume without restating. `source-reference.md` gives the actual pstack file paths behind each "inspired by" line: read those originals before building, not just this roadmap's summary of them.

Status: re-verified against the actual pstack repo (backnotprop/pstack, mirror of cursor/plugins/pstack), not written from memory. One correction from v1 noted below. Priority order remains Zoth's call, each entry stands alone, but all inherit the foundation.

**Correction from v1:** `investigate` was mapped to pstack's `figure-it-out` skill. On reread, `figure-it-out` is actually for designing a bespoke, auditable playbook for large/unusual work with no narrower playbook fit (a big migration, a multi-part change, work reviewed after stepping away), not bug investigation or hypothesis tracking. **Correction from v2, confirmed by direct read:** the hypothesis-tracking, root-cause logic doesn't live in a general `investigation` playbook either; that one turned out to cover read-only explanation requests with no hypothesis tracking in it. It lives specifically in poteto-mode's `bug-fix` playbook. `investigate` as spec'd below is still worth building, but its real reference material is `bug-fix.md` alone, not `figure-it-out` and not `investigation.md`.

Two design decisions that were open earlier in this project are now resolved; see §8 and §9 below, both marked RESOLVED rather than flagged.

**Naming note:** naming reversed from an earlier pass in this project, see `foundation.md`. No per-skill prefix; matches pstack's own convention (its skills aren't prefixed `pstack-` either) and avoids stating the `zoth-skills` namespace twice. Every skill below is referred to by its plain name now, `verify`, `clarity`, `impact`, and so on, which is the actual intended folder/file name. Scope decision (global, public distribution) still stands from before; only the naming mechanics changed. Anything Zoth-specific (ZothKit's package map, past incidents, "Zoth's voice") is still private instance data per `foundation.md` rule 5; that part is unaffected by the naming reversal.

---

## 0. dev-council: existing, not being rebuilt

Already built. Optional future integration: let it consume `impact` and `investigate` output before rendering a verdict. Not a v1 requirement.

---

## 1. impact

**Inspired by (verified):** `blast-radius`, real skill in the repo.

**Goal:** produce a Change Impact Map before a shared-boundary change is made.

**Named incident:** the R2 storage migration, a change that looked local to the storage provider but actually touched patient documents, avatars, treatment images, background processing.

**Non-goals:** not a general file dependency scanner. Not for trivial changes.

**Trigger conditions:** changes to shared/core packages in a monorepo, database schema, auth, permissions, storage, infrastructure, or public API contracts.

**Inputs:** the proposed change, and *a* project's dependency structure: a package map, a workspace manifest, a monorepo layout, a service diagram, supplied per installation, not hardcoded to one project's shape.

**Workflow:**
1. Direct dependents
2. Indirect dependents
3. Categorize: code, data/database, security/permissions, infrastructure, background jobs, tests, backwards compatibility
4. Risk rating with reasoning shown

**Output:** Impact Map: direct/indirect/data/security/operational, risk rating.

**Interface:** subagent, read-only. Reports, doesn't modify code.

**Depends on:** nothing. Buildable first, independently.

---

## 2. verify (already drafted as `runtime-proof`)

**Inspired by (verified):** `create-verification-skill`, `maintain-verification-skill`, `principle-prove-it-works`, all real skills in the repo.

**Goal:** require runtime evidence, not just static checks, before a task is marked done.

**Named incident:** the general "typecheck passes, tests pass, ship it" gap; bar already used on real work: test count and a description of what was actually run, not just asserted.

**Workflow:**
1. Define observable "done" criteria before implementation starts
2. Static checks: floor, not ceiling
3. Runtime exercise: actually run the thing
4. Edge cases from step 1's failure paths
5. Evidence block: required, explicit "not verified" lines where applicable

**Output:** Evidence block (already drafted in full).

**Interface:** skill, inline at end of implementation tasks.

**Depends on:** nothing structurally, but its incident log overlaps with `codify`/`reflect`; see resolution below.

**Status:** first draft complete (`SKILL.md` + `references/incident-log.md`), currently empty and waiting on real incidents to accumulate.

---

## 3. adversary

**Inspired by (verified):** `interrogate`, real skill: "multi-model adversarial review process."

**Goal:** attack an already-implemented change across defined attack surfaces, report findings, fix nothing itself.

**Named incident:** none yet, justified structurally (pre- vs. post-implementation review are different questions), not by a specific past failure. Worth naming a real case once or shortly after building.

**Non-goals:** doesn't modify code. Doesn't replace dev-council: dev-council asks "should we build it," this asks "did we build it correctly." Same reviewer answering both risks anchoring the post-hoc review on its own earlier design call.

**Trigger conditions:** after implementation, before merge/deploy, non-trivial changes only.

**Inputs:** the diff, relevant attack surfaces (correctness, security, data integrity, architecture, concurrency, failure handling, performance, maintainability, testing, operations; select, don't run all ten every time).

**Workflow:**
1. Select relevant surfaces for this change
2. Independent subagent reviewers per surface
3. Synthesize: critical findings, high-confidence findings, disagreements, blind spots, false positives ruled out
4. Verdict: PASS / PASS WITH CHANGES / FAIL

**Output:** Adversarial Review report.

**Interface:** parallel subagents, synthesized by a final pass.

**Depends on:** conceptually downstream of dev-council, buildable independently.

---

## 4. investigate

**Corrected reference material:** poteto-mode's `bug-fix` playbook only, not `figure-it-out`, and not the `investigation` playbook either (see correction note above). Worth having your local agent confirm this exact playbook path before building this, since two earlier references were wrong before this one.

**Goal:** force observe, reproduce, trace, hypothesize, gather evidence, then root cause, before code gets edited in response to a bug.

**Named incident:** none yet. Worth testing whether this is a problem you actually hit as sole author of most of your own code, versus a problem that matters more in unfamiliar/legacy code. Name a real case if one exists.

**Workflow:**
1. Observe the reported symptom directly
2. Reproduce it
3. Trace the actual execution path
4. List hypotheses with status (untested / disproven / confirmed)
5. Gather evidence before touching code
6. State root cause with confidence level
7. Hard rule: no code changes until sufficient evidence

**Output:** Investigation Report: hypothesis list with status, root cause, confidence.

**Interface:** subagent or inline skill; decide based on whether you want it isolated or watched live during an incident.

**Depends on:** nothing.

---

## 5. scope

**Inspired by (verified):** `poteto-mode`'s routing/scope-discipline logic, real skill.

**Goal:** define required / related / explicitly-not-required work before implementation starts.

**Named incident:** none; proceeding on a structural exception to `foundation.md` rule 4, same category as `adversary`. One-sentence structural argument, no "useful"/"helpful": this skill governs the boundaries the other skills operate within, so it needs to exist before the skills it governs are all built, not after. Ready to build.

**Workflow:**
1. State the requested outcome in the user's own terms
2. List necessary work
3. List optional/related work, flagged separately
4. List explicitly-not-required adjacent work, each with a one-line reason

**Output:** Scope statement: required / related / not-required.

**Interface:** skill, inline before implementation on any task, especially agent-delegated ones.

**Depends on:** nothing, but acts as a soft governor other skills should respect.

---

## 6. distill

**Inspired by (verified):** `unslop`'s subtraction philosophy, plus `principle-subtract-before-you-add`, real skills, applied to code rather than prose (prose is `clarity`, see below).

**Goal:** after implementation and verification, check whether the solution can get smaller without getting worse. Optimizes for minimum necessary conceptual complexity, explicitly not line count.

**Named incident:** none; checked against the structural-exception test in `foundation.md` rule 4 and doesn't qualify (no sequencing/architectural argument, just "would probably help"). Stays deferred on the plain rule. Additional reason to wait beyond the incident bar: this skill is meant to review against what `verify`'s incident log has actually caught, and that log is still empty, so building it now means it has no real signal to work from yet.

**Non-goals:** doesn't optimize for fewer lines. Runs after verify, not before, so it isn't tempted to strip something load-bearing.

**Workflow:**
1. Run only after verify has passed
2. Inspect: unnecessary abstraction, duplicate logic, unnecessary interfaces/wrappers, dead code, excessive config/error handling, needless comments, premature generalization
3. Propose specific removals with one-line justification each
4. Flag ambiguous cases rather than removing unilaterally

**Output:** Distillation Proposal: removals with justification, or "already minimal."

**Interface:** subagent, benefits from a fresh read distinct from the implementing agent.

**Depends on:** verify runs first.

---

## 7. clarity: new, built this session

**Inspired by (verified):** `unslop`, real skill; actual pattern list pulled from source, not paraphrased from memory. See full build already delivered: `SKILL.md` + `references/pattern-reference.md`.

**Goal:** review prose for AI-generated tells before publishing, engineering notebook, portfolio copy, READMEs, PR descriptions, without applying the mechanical over-correction the source skill itself contains (its literal rule is "avoid em dashes entirely," which is exactly the kind of blanket substitution Zoth has already said he doesn't want).

**Named incident:** none given; flagged as preventative (real exposure given heavy AI-agent use and public-facing writing surfaces) rather than reactive. Worth naming a specific example if one exists, to sharpen the pattern list faster.

**Non-goals:** not a grammar checker, not a voice-flattener. Doesn't touch code (`distill`'s job). Should not fire on private notes or commit messages by default.

**Structure: three tiers, not one flat list:**
- **Tier 1 (near-universal cut):** puffery, empty superlatives, AI-vocabulary tells ("delve," "tapestry"), meta-commentary. Flag directly.
- **Tier 2 (frequency-dependent, not presence-dependent):** em dashes, colons-as-connectors, rule-of-three, "it's not just X, it's Y," symmetrical parallelism. Flag as a density pattern across the piece, never ban outright.
- **Tier 3 (register-dependent, off by default in fiction):** elevated diction, heavy first-person opinion. Right for the engineering notebook, often wrong for Nicronian narrative; flagged as a question in fiction mode, not a recommendation.

**Domain scoping:** technical/professional mode (full pass), fiction mode (Tier 1 only by default, Tier 3 as question), internal/private mode (off by default).

**Output:** annotated list: passage, tier, reasoning, suggested fix. Never silently rewrites.

**Interface:** skill, manual invocation before anything public-facing goes out.

**Open decision, not yet resolved:** whether fiction mode should stay conservative by default (current setting) or run the full pass everywhere with Zoth filtering the output himself. Flagged, not decided, last time this came up.

**Status:** first draft complete this session (`SKILL.md` + `references/pattern-reference.md`).

---

## 8. codify: RESOLVED, folds into verify, not a standalone skill

**Inspired by (verified):** `principle-encode-lessons-in-structure`, real skill in the repo.

**Resolution:** codification (deciding whether a verification-failure lesson becomes a test, type, invariant, or CI check) is a step inside `verify`'s incident-log workflow. When an incident-log entry is written, the same pass asks whether it should become a new check inside `verify` itself, an `impact` category, a lint rule, or nowhere, and the entry records the answer. No separate skill or folder gets built for this.

---

## 9. reflect: RESOLVED, stays a separate skill, narrower scope

**Inspired by (verified):** `reflect` and `recall`, real skills in the repo.

**Resolution:** stays real and separate from `verify`, but scoped specifically to lessons from work that *succeeded*, a finished feature or migration that revealed something worth keeping even though nothing failed a check. If the lesson came from something that passed checks and still broke, it belongs in `verify`'s incident log instead, per the `codify` resolution above.

**Goal:** after significant work, capture what happened, what assumptions were wrong, what's reusable, and route it (temporary / documentation / project rule / test / architecture constraint / skill / nothing).

**Named incident:** the ZothKit storage abstraction turning out "too endpoint-centric" after the R2 migration, a real design insight from work that succeeded, which is exactly the category `verify`'s failure-scoped log can't hold. This is the incident that justified keeping reflect separate rather than folding it in too.

**Workflow:** the routing decision tree (temporary/doc/rule/test/constraint/skill/nothing) from the original draft, kept as-is.

**Output:** Reflection note, routed per the decision tree above.

**Interface:** skill, invoked after finishing significant work, not automatically on every task.

**Depends on:** nothing structurally. No longer blocked on the codify conflict; that's resolved above.

---

## 10. context: not ready to be a numbered skill

**Inspired by:** pstack's general context-management discipline across its subagent-spawning skills (`how`, `interrogate`, `architect` all construct scoped context for their subagents) rather than any single named skill.

**Status flag, unchanged from v1:** the source material itself describes this as shared infrastructure, not something invoked manually. Don't build it as its own skill yet; revisit once `impact`, `investigate`, and `adversary` are actually running as subagents and you can see whether context-bloat is a real, observed problem.

---

## Build order note

- `distill` runs after `verify`
- `clarity` has no dependency on anything else; buildable any time, already drafted
- `adversary` is conceptually downstream of dev-council, buildable independently
- `codify` and `reflect`: don't build both until the overlap with `verify`'s incident log is resolved
- `context`: not ready yet, revisit after the subagent-based skills are running
- `investigate`: pull the correct reference material (poteto-mode's `bug-fix` playbook only) before building, not `figure-it-out` and not the `investigation` playbook