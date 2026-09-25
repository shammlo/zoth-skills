# Surface Rubrics for adversary

One rubric per attack surface. Each reviewer receives **only its own rubric**, plus the intent and the diff.

**Select three or four. Selecting all eleven is the failure mode this file exists to prevent**. Ten reviewers each expected to produce findings will produce findings, and the real one ends up buried. Each rubric below opens with a "select this when" cue. If none of the cues match the change, that surface isn't relevant, and saying so in the output is a result.

---

## Correctness

*Select when:* almost always. This is the default surface. Skip only for changes with no behavior (formatting, comments, dependency bumps with no API change).

Does the code do what the intent says? Trace the actual paths, not the happy one. Off-by-one and boundary conditions. Empty, null, and single-element inputs. Error paths that swallow, mask, or re-raise wrongly. Return values that differ from what callers expect. Conditions that read correctly and evaluate wrongly. State that's mutated when the caller assumed a copy.

Ask specifically: what input makes this wrong? Name it concretely rather than gesturing at a category.

## Scope conformance

*Select when:* a scope statement, spec, issue, or written request exists to measure against. Without one there is nothing to conform to; say so in the output rather than reviewing against an intent you inferred yourself.

Does the diff do what was asked, and only that? Three kinds of finding, each quoting the line of the scope statement it rests on:

- **Missing or partial.** A required item with no implementation, or a partial one presented as complete.
- **Unrequested.** Behavior the scope didn't ask for, especially anything the scope statement explicitly excluded. Scope creep in a diff is a finding even when the extra code is correct, because it widens what has to be reviewed and verified.
- **Implemented, but not as specified.** The item exists and does something other than what the scope describes.

This is the one surface that can fail a change whose code is flawless. A correct implementation of the wrong thing passes every other rubric in this file, which is why it is kept separate from correctness rather than folded into it.

## Security

*Select when:* the change touches auth, sessions, permissions, roles, tenant scoping, user input, file paths, credentials, tokens, external requests, or anything that renders untrusted content.

Authorization checked at every entry point, not just the obvious one. **Verify the negative case**, that the wrong role, wrong tenant, or wrong owner is actually rejected. A test proving the right user succeeds proves nothing about the check. Injection paths (SQL, command, path traversal, template). Secrets in logs, errors, or responses. Tokens with wrong scope or lifetime. Input trusted because it arrived from an internal caller that doesn't validate it either.

Security failures are usually silent: the wrong answer looks like a normal response.

## Data integrity

*Select when:* the change writes, migrates, deletes, or changes the shape of stored data, including schema changes, background writes, and anything with a unique constraint.

What happens to rows already written under the old assumption. Partial writes and interrupted operations. Transaction boundaries, and whether the unit of work is actually atomic. Duplicate or retried operations that double-create. Cascading deletes reaching further than intended. Migrations that can't be rolled back, or that lock a table long enough to matter. Constraints enforced in code but not in the database.

Ask: if this ran twice, or halfway, what would the data look like?

## Architecture

*Select when:* the change adds an abstraction, crosses a module or service boundary, introduces a dependency direction, or is the second implementation of something.

Does this fit the structure that exists, or fight it? Dependencies pointing the wrong way. Business logic leaking into transport or persistence layers. Abstractions introduced for one caller. Duplicated concepts that will now drift apart. Coupling added where a boundary existed.

Distinguish "different from how I'd do it" from "will cause a problem." Only the second is a finding.

## Concurrency

*Select when:* the change touches shared mutable state, caching, background jobs, queues, request-scoped state reused across requests, or anything with a read-then-write.

Race conditions between read and write. Assumptions that a value hasn't changed since it was read. Locks acquired in inconsistent order. Non-atomic check-then-act. Shared state across requests or workers. Retries running concurrently with the original. Ordering assumed but not guaranteed by the queue.

Skip this surface honestly when the change is genuinely single-threaded and single-writer. Speculative concurrency findings are the most common form of review noise.

## Failure handling

*Select when:* the change calls anything that can fail (network, database, filesystem, external service), or adds retry, timeout, or fallback behavior.

What happens when the dependency is down, slow, or returns something unexpected. Timeouts present and sized deliberately. Retries that are safe to repeat, with backoff. Errors that lose the original cause. Failures that leave state half-updated. Fallbacks that mask an outage until it's much worse. Errors logged at a level nobody reads.

## Performance

*Select when:* the change adds queries to a loop, touches a hot path, processes unbounded input, or changes an algorithm's complexity.

Queries inside loops. Complexity that scales with data the change doesn't control. Unbounded result sets, memory growth, or recursion. Indexes the new query needs and doesn't have. Payloads that grow with usage. Work done per-request that could be done once.

**Only report performance findings with a reason to believe the scale matters here.** "This is O(n²)" is not a finding without an argument about what n is. Premature optimization dressed as review is still premature.

## Maintainability

*Select when:* the change will be extended by someone else, or is complex enough that the next reader's understanding is at risk.

Will the next person understand this? Names that mislead rather than merely disappoint. Logic whose reason for existing isn't recoverable from the code. Comments contradicting the code. Special cases with no explanation. Configuration that must be kept in sync by hand.

Lowest-severity surface by default. Findings here rarely block a merge, and framing them as though they do costs the review credibility on the findings that do.

## Testing

*Select when:* almost always alongside correctness, but focused on the reach of the change, not the file that changed.

Does coverage exist over the behavior that changed, and over what it reaches? Tests asserting the implementation rather than the behavior, and which will fail on a valid refactor. Failure paths untested while the happy path is covered three times. Tests that pass whether or not the code works. Mocks so thorough the test proves only that the mock was called.

The gap that matters is untested *reach*, not untested lines.

## Operations

*Select when:* the change affects deploy, configuration, migrations, feature flags, monitoring, or anything whose failure would need to be noticed in production.

Is this observable when it breaks? Would anyone find out from monitoring rather than from a user? Deploy order requirements, and whether anything enforces them. Rollback: is it possible, and does it work with the data written meanwhile. Configuration differing across environments. Feature flags with no removal plan. New failure modes with no alert.
