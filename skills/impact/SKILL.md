---
name: impact
description: "Maps what a change to a shared boundary reaches before it is made: direct and indirect dependents, categorized, with a reasoned risk rating. Use before changing shared packages, database schema, auth, permissions, storage, infrastructure, or a public API contract, or when a change calls itself \"small\" but touches something other code depends on. Reports only, never modifies code. Skip trivial changes."
---

# Impact

Produce a Change Impact Map *before* a shared-boundary change is made, so the decision to make it is taken with its reach visible rather than discovered afterward.

The motivating problem class: a storage-provider migration that looked local to one module turned out to touch user documents, avatars, uploaded images, and background processing, none of which appeared in the change's own description. Nothing about that is specific to one project. Any change to a boundary other code depends on has a reach larger than its diff, and the reach is invisible from inside the module being changed.

**Listing the callers is not the job.** An agent can grep those in seconds, and a list of call sites is the part you'd have found anyway. The job is the reach grep won't show you: the JSON shape an API already returns, the column another service reads, the queue message a worker in a different language parses, the cached value with a different lifetime, the consumer three hops downstream that never names the symbol you're changing.

## When this fires

Before implementation, on changes to:

- Shared or internal packages that multiple applications import
- Database schema, including columns, constraints, indexes, enum values, nullability
- Auth, sessions, permissions, or role/tenant scoping
- File storage, object storage, bucket layout, or URL/visibility semantics
- Infrastructure, deployment topology, environment configuration
- Public API contracts, including routes, request/response shapes, error codes, webhook payloads

**Non-goals.** This is not a general file dependency scanner; if the answer is a list of imports, this skill has nothing to add. It is not for trivial changes. A change contained inside one module with no external consumers doesn't need a map, and running it there trains you to ignore the output. It does not modify code, propose a fix, or write the change. It reports, and the decision stays with whoever asked.

## Inputs

Give it the change and the project's own dependency structure. Specifically:

- **The proposed change.** Stated as intent, not as a diff, since the diff doesn't exist yet
- **The project's dependency structure.** However that project expresses it: a package map, a workspace manifest, a monorepo layout, a service diagram, a migration plan. This is taken as input, not assumed. A skill that hardcodes one project's package list works for exactly one project.
- **The constraints that apply.** A phased migration already in progress, a compatibility window, a deploy order, a tenant isolation guarantee

If any of these is missing, ask for it or state its absence in the output. Do not infer a dependency structure from directory names and present it as fact.

## Workflow

### 1. Direct dependents

What imports, calls, queries, or reads the thing being changed. This is the grep-able layer. Get it complete, cite real `file:line` locations, and move on quickly. It's the floor.

A search that returns nothing is still a finding. "No consumers outside this package, confirmed by searching for X, Y, and Z" is a useful result, not an empty one.

### 2. Indirect dependents, where the real answer is

Trace outward from step 1. Spend most of the time here.

- Consumers of the consumers, until the chain stops mattering
- Data-shape coupling: an API response another client already parses, a column another query reads, a serialized payload another process deserializes
- Contract coupling with no symbol reference at all: a wire format, a file naming convention, a URL another system constructs by hand, a value another language reads from the same bytes
- Timing coupling: caches, retries, background jobs, anything holding the old shape while the new one deploys
- Configuration coupling: feature flags, environment variables, deploy-order assumptions

See `references/surface-checklist.md` for the full list of places to look, and for what each of the seven categories in step 3 actually covers.

### 3. Categorize

Sort every finding into these, and say plainly when a category is empty:

- **Code.** Call sites, imports, type dependencies
- **Data / database.** Schema, migrations, existing rows, queries reading the changed shape
- **Security / permissions.** Auth checks, role or tenant scoping, anything where a wrong answer leaks data rather than erroring
- **Infrastructure.** Deploy, config, environment, external services
- **Background jobs.** Workers, cron, queues, webhooks, anything running outside the request that made the change
- **Tests.** What coverage exists for the reach, and where there is none
- **Backwards compatibility.** What breaks for clients on the old shape during and after the change

An empty category stated explicitly is information. An empty category silently omitted reads as "not applicable" when it often means "not checked."

### 4. Risk rating, with the reasoning shown

Give the change a rating, and give the reasoning that produced it in the same breath. The reasoning is the deliverable; the label is shorthand for it.

- **LOW.** Reach is contained, consumers are all in-repo and covered by tests, a mistake surfaces as a loud failure
- **MEDIUM.** Reach extends past the immediate package, but every consumer is identified and a mistake is visible
- **HIGH.** Reach includes a category where a mistake is silent rather than loud: data written wrong, a permission check that passes when it should fail, a job that stops running without erroring
- **CRITICAL.** Reach includes a silent-failure category *and* something in that reach could not be verified

Do not compute the rating from a count of findings. Twelve trivial call sites is a LOW; one unverified permission boundary is not. Two changes with the same finding count can land three levels apart, and the reasoning is what tells them apart. If a rating feels forced, say which way it's ambiguous and why rather than rounding.

## How sure are you, and where each fact stopped

The map's usefulness depends on facts that are easy to assert and harder to establish. For each fact the assessment rests on, get it as far down this ladder as is cheap, and **state where it stopped**:

1. **Asserted.** You said so. Worth nothing on its own.
2. **Cited.** A real `file:line`, or the dependency's own source.
3. **Reasoned through.** You walked the failure path and showed it can't reach.
4. **Executed.** You ran something against the current system that would fail loudly if you were wrong.

Level 4 looks different here than in a post-change review. This skill runs *before* the change exists, so you cannot run the changed code. What you can run is a check against the system as it stands: query the column to see whether it's actually nullable in practice, list the objects in the bucket to see the real key layout, call the endpoint to see the shape it actually returns. Those are level 4 facts about the ground the change lands on.

Any fact that stops at level 1 goes in the output as unverified. Never round up, and never present an inference as an observation. Do not name a consumer, an endpoint, or a column you have not actually seen.

## Output

Follow the shared shape. Findings, then explicit gaps, then verdict:

```
IMPACT: [the proposed change, in one line]

Findings:
  Code:
    - [consumer, file:line, what breaks and why, confidence level]
  Data / database:
    - [finding, with reasoning]
  Security / permissions:
    - [finding, with reasoning]
  Infrastructure:
    - [finding, or "none found, searched X, Y"]
  Background jobs:
    - [finding, with reasoning]
  Tests:
    - [what covers the reach, and what doesn't]
  Backwards compatibility:
    - [what breaks for old clients, and for how long]

  Cleared:
    - [things checked and found not to be affected, with what was checked]

Not checked / not applicable:
  - [each gap on its own line, with why it couldn't be checked]

Verdict:
  Risk: [LOW | MEDIUM | HIGH | CRITICAL]
  Reasoning: [what drove the rating, naming the specific finding that set it]
  Before making this change: [the cheapest check that would catch the real problem]
```

`Cleared` and `Not checked` are different and both required. Cleared means you looked and it's fine. Not checked means you didn't look, or couldn't. Collapsing them is the failure this skill exists to prevent.

## Interface, run this as a subagent

Read-only, isolated context, returns the map. The tracing in step 2 generates a lot of intermediate reading that has no value in the main conversation, and doing it inline buries the decision under the search.

Give the subagent the task, the specifically relevant files and contracts, and the specifically relevant constraints. Not the whole repository by default. If it turns out to need more, it can ask. That's cheaper than starting from everything.

## Interaction with other skills

Runs **before** the change, where a post-implementation review runs after; the two ask different questions and shouldn't be collapsed into one reviewer. If this skill flags consumers of a changed module, `verify`'s evidence block should later include runtime confirmation that those specific consumers still behave correctly. This skill names what to verify, and `verify` is where the verification gets recorded.
