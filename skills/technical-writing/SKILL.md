---
name: technical-writing
description: Structure and plain sentences for writing an engineer acts on, end-of-task reports, PR descriptions, commit messages, docs, RFCs. Use when drafting or reviewing any of these, including your own report at the end of a task. Pair with clarity, which owns voice.
---

# Technical writing

The goal: a tired engineer understands it on the first read and knows what to do next.

Three rules sit above everything else:

- **Cut every word that does no work.** "In order to" is "to". "It is important to note that" is nothing.
- **Use the short, everyday word.** "Use", not "utilize". "Do", not "perform".
- **When a rule makes a sentence worse, fix the sentence another way or leave it.** The rules serve the reader. A sentence that obeys every rule and reads like a machine wrote it has failed.

The codebase is the word list. Write the real symbol, file, flag or command, not a description of it. Don't invent jargon; say what a developer would say out loud.

## Reports and PR descriptions

These are the writing an agent produces most, and where length costs the most.

**End-of-task report to the user.** In this order, and nothing else:

1. **Outcome** — one or two sentences: what changed and where (link the PR).
2. **What the reader must know** — decisions made on their behalf, behaviour that changed, surprises. Only what changes what they do or believe.
3. **Checks** — what ran and passed, what was deferred, what was not verified. A deferred check is stated, never implied to have passed.
4. **What they need to do** — merges, approvals, commands to run. Numbered if more than one.

Leave out: the narrative of how you got there, files read, dead ends, restating the request, a closing offer of more help. If a detail only matters to someone reviewing the diff, it belongs in the PR, not the report.

**PR description.** A briefing a reviewer reads in under a minute: why, what changed (grouped by area, not by file), how it was verified, what was deferred, anything the reviewer must decide. Link logs and long tables instead of pasting them. Follow the repo's PR template when one exists.

**Commit message.** Subject says what the commit does. Body says why, and anything non-obvious a future reader of `git log` needs.

## Pick the mode first (longer docs)

One document, one mode. Does the reader need to *do* something or *understand* something, and are they learning or working?

- **Tutorial** (doing, learning): says what the reader will build; every step produces a visible result; explanation cut to a clause and a link.
- **How-to** (doing, working): solves one problem the reader has; assumes competence; action only. Name it by the task.
- **Reference** (understanding, working): facts, options, limits, errors. Dry, complete, no opinion. Mirrors the structure of the thing it describes.
- **Explanation** (understanding, learning): one topic, the why — decisions, constraints, alternatives. Opinion belongs here and nowhere else.

Don't mix modes in one document. Split and link.

## Sentences

- Address the reader as "you", in the present tense.
- Say who does what: "the guard rejects", not "is rejected". Passive only when the actor is unknown or irrelevant.
- Instructions are commands: "Run `pnpm build`." Put the condition first: "To re-vendor, run …".
- Common case first, exceptions after.
- One instruction per sentence; one thought per sentence elsewhere. Split sentences past about 25 words — but keep a long sentence that carries one thought with its condition.
- Vary length on purpose. All-short reads clipped and mechanical.
- Be specific: not "schema changes can cause issues" but "a column rename fails the build".
- Headings carry the point ("Build before you test"), in sentence case.
- Numbered lists for sequences, bullets otherwise. Introduce a list with a sentence; keep items parallel.
- Code in code font. No "simply", "easy" or "just" in instructions.

## No sentence with two readings

- Keep "only" and "not" next to the word they change.
- Break up long noun strings: "the vendor tarball hash verification step" → "the step that checks the tarball hashes".
- Every "it", "this" and "they" points at one obvious thing. Repeat the noun when in doubt.
- Don't drop verbs: "Phase 1 moves the converters and Phase 2 the runtime" leaves Phase 2 without one.
- Call each thing by one name everywhere. Three names for one thing teaches three things.
- No slashes for "and/or": write "a, b, or both". No "(s)" plurals.
- Skip idioms, Latin abbreviations and metaphors. Non-native readers and agents parse plain constructions best.
- Punctuation (em dashes, semicolons, parentheses) is judged by frequency and fit, per `clarity`, not banned.

## Worked example

Before:

> I went ahead and took a look at the audit service and it turned out that there were a number of places where the diff was being computed manually, so I refactored those to use the new helper, which should make things more consistent going forward. I also ran the tests and they seem to be passing.

After:

> Audit entries now go through `logChange`; the six hand-built diffs are gone. Tests: `audit`, `roles` and `patients` specs pass (304). Not run: the web app in a browser.

## Source

Reworked from pstack's `technical-writing` (Lauren Tan, MIT), which layers Diátaxis, Google developer style, ASD-STE100 and Global English. Changes: model-invoked instead of manual; a section for agent reports; punctuation deferred to `clarity` rather than banned; `unslop` replaced by `clarity`. See `source-reference.md`.
