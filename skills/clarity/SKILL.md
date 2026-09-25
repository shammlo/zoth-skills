---
name: clarity
description: Removes AI-generated tells from prose. On public-facing writing (READMEs, portfolio copy, notebook entries) it flags and proposes rewrites, never rewriting silently. On an agent's own end-of-task report or PR description it applies the fixes while drafting. Judges by frequency and context, never by blanket bans. Skips commit messages, private notes and fiction unless asked.
---

# Clarity

Reviews writing for AI-generated tells and proposes fixes, but never at the cost of turning good writing mechanical. See `references/pattern-reference.md` for the full tiered pattern list before running a pass.

## The rule this skill can't violate

No absolute bans. Every flag is "here's a pattern, here's why, here's a suggested fix", never a silent rewrite, never a rule shaped like "never use X." The one exception is the agent's own draft in report mode (below): there is no one to propose to, so it applies the fixes while writing. pstack's original `unslop` skill includes "avoid em dashes entirely" as a hard rule; this version deliberately does not carry that over. Punctuation and rhetorical choices get judged by frequency and fit, not banned by category.

## Domain scoping, decide this before running

Ask, or infer from context, which mode applies:

**Technical/professional mode** (engineering notebook, portfolio, READMEs, docs): apply the full pattern reference normally. Puffery, empty superlatives, and AI vocabulary are almost always worth cutting here. First-person voice and stated opinion are usually strengths, not tells, in this register.

**Fiction mode** (a personal fiction project, whatever that looks like for the installer): Tier 3 patterns are off by default. Elevated diction, ceremonial phrasing, and structural repetition may be deliberate stylistic choices serving character voice or mythic register, not AI tells. Only run Tier 1 (true filler/empty-superlative cuts) unless the user explicitly asks for a full pass. When in doubt on a fiction passage, flag it as a question rather than a recommendation: "this reads either as intentional elevated register or as a tell. Which is it?"

**Report mode** (an agent's own end-of-task report, PR descriptions): apply Tier 1 and Tier 2 while drafting, silently, with no findings list. These are read by the owner on every task, so filler, hedging and puffery cost the most here. Structure comes from `technical-writing`; this skill only removes the tells.

**Internal/private mode** (commit messages, working notes): this skill should not fire here by default. If asked to run anyway, treat it as technical mode but hold every flag to a higher bar. Internal writing doesn't need the same polish as anything public-facing.

## Workflow

1. Confirm or infer the domain (above) before scanning anything
2. Read the full piece once for content, don't flag mid-read
3. Scan against `references/pattern-reference.md`, tier by tier
4. For Tier 1: flag directly, propose a specific rewrite
5. For Tier 2: report as a pattern across the piece ("X appears N times"), not per-instance flags, and only if density is actually high
6. For Tier 3: only in technical mode by default; in fiction mode, only flag as a question, never a recommendation
7. Present findings as an annotated list (passage, tier, reasoning, suggested fix) and stop. The user decides what changes, this skill doesn't publish or auto-edit.

## Output format

```
Flagged (Tier 1, recommend cut):
"...this represents a pivotal moment for the platform..."
→ Puffery, no specific claim underneath. Suggest: state what actually changed.

Pattern (Tier 2, density check):
"...the rollback was clean — three commands — and the incident closed in
under an hour..."
Em dash used 5 times across 3 paragraphs, same aside-construction each time.
→ Not individually wrong, but the repetition reads as a tell. Consider varying
  the construction in 2-3 of these.

Question (Tier 3, fiction mode, not auto-flagged):
"...a testament to the elder's restraint..." (formal third-person narration)
→ Could be intentional elevated register for the character, or a tell.
  Flagging for your judgment, not proposing a change.
```

## Calibration

This skill gets better specifically by learning the installer's actual voice, not a generic one. If a Tier 2 or Tier 3 flag turns out to be wrong more than once (something genuinely the writer's style getting mistaken for a tell), note it. Over time the pattern reference should get a "confirmed voice, stop flagging" list specific to constructions this writer actually uses on purpose. This calibration list is per-installer, local, and never ships pre-filled. Someone installing this skill starts with an empty calibration list that fills in from their own corrections, not from anyone else's.
