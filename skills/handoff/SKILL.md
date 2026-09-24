---
name: handoff
description: Condenses the current session into a short handoff note a fresh session can resume from, pointing at existing artifacts instead of copying them, separating verified state from assumptions, and recording decisions with their reasons so they aren't re-argued. Run manually with /handoff, optionally naming what the next session will focus on.
argument-hint: "What will the next session focus on?"
disable-model-invocation: true
---

# Handoff

A long session gets more expensive every turn, because every turn resends everything before it. Starting fresh is the cheapest fix, but only if the fresh session doesn't have to rediscover what this one already learned. This skill writes the note that makes starting fresh safe.

The note is not a transcript summary. It's the minimum the next session needs to act correctly: where things stand, what was decided and why, what's still open, and what to check before trusting any of it.

## When to use it

- The session is long and the next piece of work doesn't need most of what's in it.
- You're switching to a different task and will come back to this one later.
- Work is moving to another agent, another machine, or another person.
- Context is close to being compacted, and you'd rather choose what survives than let a summary choose for you.

If the user named a focus for the next session, shape the note around it and leave out threads that don't bear on it. Mention them in one line so they aren't lost silently.

## Writing the note

### 1. Point, don't copy

Anything that already exists somewhere durable gets a pointer, not a restatement: commits, branches, PRs, issues, specs, scope statements, design docs, files in the repo. The next session can open a path. Copying an artifact into the note doubles its cost and creates a second version that can go stale. The note holds only what exists nowhere else: the reasoning, the dead ends, the current mental model.

### 2. Separate what's verified from what's assumed

Every claim about state is one of two things, and the note says which:

- **Verified.** Checked in this session, with how: "tests pass, ran `<command>`", "PR open, checked on the host".
- **Assumed / not checked.** Believed but not confirmed, or confirmed long enough ago that it may have drifted.

A fresh session trusts whatever it's handed. An unverified claim written as fact becomes the next session's first bug.

### 3. Record decisions with their reasons, and the options rejected

This is what gets lost most often and costs the most to rebuild. For each decision that shaped the work: what was chosen, why, and what was considered and rejected, in a line each. Without the rejected options, the next session rediscovers them, finds them plausible, and reopens a question that was already settled.

### 4. End with one next action

Not a list of possibilities. The single most useful next step, concrete enough to start on immediately. Other open threads go in their own section.

### 5. Keep it small

The note should be a small fraction of the session it replaces. Roughly a screen is a good target for most work. If it's growing well past that, something is being copied that should be pointed to. Cut detail before cutting threads: a thread the next session doesn't know about is worse than one it knows about briefly.

### 6. Redact

Strip credentials, tokens, keys, personal data, and anything the user marked private. The note may be pasted somewhere with a wider audience than this session had.

## Where the note goes

Choose based on where the next session will run, and say where the note went:

- **Same machine, later.** Write it to the system's temp directory (`$TMPDIR`, falling back to `/tmp`, or `%TEMP%` on Windows) with a descriptive, timestamped filename. It stays out of the repo, so nothing gets committed by accident.
- **Somewhere else, or an ephemeral environment** (a cloud container, CI, a remote session that will be reclaimed). A temp file won't survive. Give the note in the reply so the user can carry it, or, if the user agrees, save it where the next session can reach it. Don't commit a handoff note to a shared branch without asking.
- **The user named a location.** Use it.

If the host can start a new session seeded with a prompt, offer that as an option. The note becomes the prompt, and redaction matters even more.

## The note

```
HANDOFF: [what this work is, one line]
Written: [date]  Next focus: [what the next session is for, if given]

Resume instructions for the next session:
  Read this note, run the checks under "Verify on pickup", and report any
  mismatch before acting on anything below.

State:
  - [branch / PR / key files, as pointers]  (verified: how | assumed)

Decisions:
  - [decision]: [why]. Rejected: [option], [why not]

Dead ends:
  - [approach tried]: [why it didn't work]

Open threads:
  - [thread, one line each]

Verify on pickup:
  - [command or check that confirms the state above is still true,
     e.g. branch exists and is clean, PR still open, tests still pass]

Suggested skills:
  - [only skills installed in this session, and why each applies]

Next action:
  [one concrete step]
```

Drop any section that has nothing in it, except "Verify on pickup" and "Next action", which are always present. A handoff with nothing to verify is claiming the world didn't change between sessions.

## Interaction with other skills

- **`context`** keeps a single session lean. This is for when a session has done its job and the work should continue in a fresh one.
- **`scope`**: if a scope statement exists, point to it rather than restating the boundaries.
- **`verify`**: an evidence block written earlier in the session is the best source for the "verified" lines. Point to it.
- **`reflect`** captures lessons worth keeping beyond this piece of work. A handoff is about continuing this work; if something in it deserves to outlive the task, route it through `reflect` instead.
