---
name: context
description: Keeps the main conversation's context spent on what changes the next decision. Use before reads or commands likely to return a lot (whole large files, full logs, broad searches, generated code), before spawning subagents, before re-reading something already seen, and when asked why a session or skill set is burning tokens. Reads selectively, routes bulk to subagents that return summaries, and briefs them with pointers instead of payloads.
---

# Context

The context window is finite and every token in it is paid for again on each turn that follows. This skill exists so that what fills it is the material the next decision depends on, not raw output that was read once and never used.

It is a working discipline, not a report generator. Most of the time it changes *how* you read and delegate, and produces nothing visible. The exception is the audit in the last section, run when someone asks where their tokens are going.

## When this fires

- Before a read or command whose output is likely to be large relative to what you need from it: a whole file of several hundred lines, a full test or build log, a broad grep, a lockfile, generated code, a long diff.
- Before spawning one or more subagents.
- Before re-reading a file or re-running a command whose result is already in the conversation.
- When a long session is heading toward compaction.
- When asked why a session, a workflow, or an installed set of skills costs so much.

## The workflow

### 1. Name the question before reading

Before opening something large, say what you need from it. Then read the part that answers it: a line range, a grep with a few lines of context, `head`/`tail`, `git diff --stat` before the full diff, failures only from a test run. If you can't name the question, you're browsing, and browsing belongs in a subagent (step 2) or doesn't happen.

This is a judgment call, not a size threshold. A 400-line file you're about to edit throughout is worth reading whole. A 60-line file you need one constant from is not.

### 2. Route bulk away from the main thread

When answering needs a lot of raw material (many files, a long log, a sweep across a subsystem), delegate it to a subagent that returns a summary with `file:line` pointers. The raw material stays in the subagent; the main thread gets what it needs to decide.

Delegation isn't free. A subagent starts cold and re-reads whatever it needs, so for one known file, reading the relevant region directly is cheaper than briefing someone else to. Delegate when the raw material is much larger than the answer *and* you won't need the raw material again. If you will need it again (you're about to edit it), read it yourself.

### 3. Brief subagents with pointers, not payloads

A subagent brief contains the task, the specifically relevant paths, the constraints, the output shape, and a length cap. It doesn't contain pasted file contents or a replay of the conversation. The subagent can open a path; it can't un-read a pasted file it didn't need.

Pick the model for the role where the host allows it (in Claude Code, the Agent tool's `model` parameter). Mechanical searching, collecting, and summarizing go to the cheapest model that does them reliably. Judgment (a synthesis, a verdict, a design call) stays on the strongest one. Say which you chose when it isn't the default.

### 4. Don't re-derive what's already established

Before re-reading a file or re-running a command, check whether the answer is already in the conversation and whether anything could have changed it since. If nothing did, use what you have. Re-reading "to be sure" is the most common silent cost in a long session.

### 5. Decide fan-out size before spawning

Choose the number of parallel subagents from the task, before launching any, and state it. Each one pays the cold-start cost. More reviewers produce more findings, not more signal (see `adversary`'s surface-selection step for why). If a first pass answers the question, don't launch a second one to confirm what nobody doubts.

### 6. Summaries keep the evidence

Guarding context never means dropping the proof. When you filter or summarize output, keep failures, error messages, and the specific lines a conclusion rests on verbatim. A summary that says "tests mostly pass" in place of the three failing test names has saved tokens by destroying the evidence `verify` needs. Cut the passing noise, keep the failing signal.

## Auditing a skill set

When asked why an installed set of skills or a workflow costs so much, check the places cost hides and report in the shared shape:

- **Always-loaded text.** Every model-invocable skill's description is in context in every session, whether or not the skill runs. Measure them. Long trigger-phrase lists are the usual culprit.
- **Accidental triggers.** A description that matches common phrasing ("done", "review this") loads the skill body, and whatever it does next, far more often than intended.
- **Fan-out skills.** Skills that spawn several subagents are the expensive ones per run. Check whether they can auto-trigger, and whether they set a model per role or run every subagent on the strongest model.
- **Duplicates.** The same skill installed twice (for example, once as a personal skill and once through a plugin) pays its description cost twice.
- **Reference files read on every run.** Content needed on every invocation belongs in `SKILL.md`; `references/` is for what's needed sometimes. A reference file read every time costs a tool round-trip for no benefit.

```
CONTEXT: [what was audited]

Findings:
  - [where the cost is, with a measurement or the specific trigger, and why it matters]

Not checked / not applicable:
  - [e.g. actual per-session token usage, if no usage data was available]

Recommendation:
  - [specific changes, most savings first: shorten X, make Y manual-only, set a model per role in Z]
```

Measured sizes beat estimates; say which you have. File size divided by about four is a fair token estimate for English prose, and should be labeled as an estimate.

## Why this exists

Deferred on the roadmap until context cost was an observed problem rather than a hypothetical one. It became one: a report of heavy token use, followed by an audit of this repo's own skills, found roughly 4.7 KB of descriptions loaded into every session, the two fan-out skills (`dev-council`, `adversary`) able to trigger automatically, and `verify` triggering on the word "done". Those fixes shipped alongside this skill. This skill is the discipline that keeps the same problems from coming back.

## Interaction with other skills

- **`impact`, `adversary`, `dev-council`:** their subagent briefs follow step 3. Their fan-out size follows step 5.
- **`verify`:** step 6 exists for it. Filter output, never the evidence.
- **Skill authors:** the audit section doubles as a checklist before shipping a new skill: a short description, manual-only if it fans out, a model per role, inline content for what's used every run.
