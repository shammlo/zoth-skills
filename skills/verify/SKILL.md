---
name: verify
description: Requires runtime evidence, not only static checks, before a feature, fix, or migration is reported done. Use when closing out non-trivial implementation work, especially changes to auth, permissions, payments, file storage, multi-tenant queries, background jobs, or a shared package, and whenever a completion summary lists only typecheck, lint, or unit tests. Skip docs-only edits, config tweaks, and one-line changes.
---

# Verify

Static checks passing is not evidence the feature works. An agent that reports "typecheck clean, tests green" has told you the code compiles and the tests it wrote pass, not that the thing behaves correctly when run. The bar here: every "done" claim carries both the test count and a description of what was actually executed at runtime, on every task, not just the ones where it happened to be top of mind.

## When this fires

Before accepting or writing a "this is done" summary for non-trivial implementation work, whether from a subagent, an AI coding tool, or your own end-of-task report. Docs-only edits, config tweaks, and one-line changes don't need the full workflow; a sentence saying how the change was checked is enough. Applies most forcefully to:

- Anything crossing a shared/core package boundary in a monorepo (auth, permissions, file handling, audit logs, translations, or whatever your project's shared layer covers)
- Auth and permission checks. Did you confirm a request from the *wrong* role/tenant is actually rejected, not just that the right one succeeds?
- Payments, billing, or any operation that creates a record as a side effect. Did you confirm retries/duplicate requests don't double-create?
- File uploads and storage (bucket visibility, signed URLs). Did you confirm the URL/visibility is correct for the actual storage backend, not assumed from the code path?
- Multi-tenant or scoped queries. Did you confirm data from one tenant doesn't leak into another's view, not just that the happy path returns data?
- Background jobs, webhooks, cron. Did you confirm the job runs and produces the expected side effect, not just that the handler function typechecks?

## The workflow

### 1. Define "done" before starting, not after

Before implementation begins, write down, in a sentence or two, what observable behavior proves this is finished. Not "the endpoint exists" but a specific, checkable claim, for example "a request from an unauthorized tenant returns 403 and no row is written." If you can't state this up front, the task isn't scoped enough to verify later.

### 2. Static checks are the floor, not the ceiling

Typecheck and lint clean, and the relevant unit/integration test suite passes. This is necessary and non-negotiable, but insufficient on its own. Never report a task as done on the strength of this step alone.

### 3. Runtime exercise, the step agents skip

Actually run the thing. This means one of:
- Hit the endpoint/flow with a real (or realistic local) request and read the actual response, not the code that generates it
- Run the migration or job against a real (test/local) database and inspect the resulting rows
- For UI work, describe what was actually rendered/clicked, not just that the component compiles

If there's no way to exercise it locally (e.g. it depends on an external service you don't have credentials for), say so explicitly in the evidence block below rather than silently skipping this step.

### 4. Edge cases from the "done" definition in step 1

Test the failure paths you named, not just the happy path. A feature that only proves the authorized-user case works is not proof the authorization check works.

### 5. Evidence block, required output

Every task closes with a short block like this, not a bare "done":

```
Evidence:
- Typecheck: clean
- Tests: 42 passing (12 new: auth-boundary, tenant-isolation, upload-visibility)
- Runtime: hit POST /api/documents locally with a cross-tenant auth token,
  got 403, confirmed no row written to the documents table
- Not verified: production storage bucket visibility (no prod credentials in
  this session). Flagged for manual check before deploy
```

If any line would honestly read "assumed, not checked," that line goes in the evidence block explicitly. It does not get silently omitted.

### 6. Codify, when this task's evidence surfaces a lesson, decide where it goes

This step only fires when step 3's runtime exercise catches something that passed the static checks in step 2 and still would have shipped broken, the specific failure category this skill exists to close. When that happens, before the incident-log entry (below) is written, decide: does this lesson become a new check inside this skill itself (a permanent addition to what gets exercised next time), a new category in `impact`'s checklist, a lint rule, or does it stay a one-off entry with no structural change? Record the answer in the incident-log entry's "Codified as" field. If the honest answer is "nothing, this was a one-time fluke," that's a valid answer. Write it down rather than skipping the field.

This is deliberately narrow: it only applies to lessons from verification failures. A design insight from work that succeeded (nothing broke, nothing failed a check, but something worth remembering came out of it anyway) belongs in `reflect`, not here.

## Known failure modes (fill this in as it happens)

`references/incident-log.md` accumulates real incidents from your own projects, so future runs check for what has actually bitten you rather than a generic list. Add an entry whenever something passed typecheck and tests and still shipped broken. It starts empty for every installer.

## Interaction with a design-review skill and an impact-mapping skill

This skill runs *after* implementation, as the last gate before "done." It doesn't replace a design review before implementation (`dev-council`) or an impact map of what a shared-boundary change reaches (`impact`). If an impact map flagged consumers of a changed module, this skill's evidence block should include runtime confirmation that those specific consumers still behave correctly, not just that the module's own tests pass.
