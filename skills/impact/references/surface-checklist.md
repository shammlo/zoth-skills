# Surface Checklist for impact

Loaded on demand during step 2 (indirect dependents) and step 3 (categorize). Two lists: where to look when grep stops, and what each category actually covers.

**None of this is a required checklist to run end to end.** Most changes touch three or four of these. Working through all of them mechanically produces a long map with a low signal ratio, which is its own failure. A map nobody reads protects nothing. Pick the surfaces the change actually plausibly reaches, and say which ones you skipped and why.

---

## Where grep stops

The call sites are the easy half. These are the couplings that exist without any symbol reference to search for:

**Shape coupling, something parses what you produce**
- The JSON an endpoint already returns, parsed by a client you don't control
- A database column another query reads, or another service reads directly
- A serialized payload (queue message, cache entry, session blob) written by one version and read by another
- A file format or wire format read by code in a different language
- Anything with a schema written down in two places that must agree

**Name and path coupling, something constructs a reference by hand**
- A URL built by string concatenation rather than from a shared helper
- An object storage key layout, a bucket path convention, a filename pattern
- A route, a queue name, a topic, an env var name referenced as a literal
- An external system that has your old path saved in its own config

**Timing coupling, old and new coexist**
- Caches holding the previous shape past the deploy
- In-flight requests and retries started before the change and landing after
- Background jobs, cron, queued work created under the old contract
- Deploy order: what if the consumer ships before the producer, or after
- Migrations that run while the old code is still serving

**Configuration coupling**
- Feature flags gating either side of the change
- Environment variables differing between local, staging, and production
- Anything configured per-tenant or per-environment rather than in code

**Distance coupling**
- Consumers three or more hops downstream that never name the changed symbol
- A dependency's own source: does the version actually pinned behave the way the docs say
- Local patches or overrides applied to a dependency

---

## What each category covers

**Code.** Call sites, imports, type dependencies, subclasses and implementations, anything that fails to compile or resolve. The loudest category and therefore the least dangerous.

**Data / database.** Schema changes (columns, types, nullability, constraints, indexes, enum values). Existing rows written under the old assumption. A column that becomes non-null has to answer for every row already there. Queries reading the changed shape, including ones in migrations, reports, and admin tooling. Ask what happens to data already at rest, not just data written after.

**Security / permissions.** Auth checks, session handling, role scoping, tenant or organization isolation, ownership checks, object visibility. This category earns its own line because its failures are silent: a permission check that wrongly passes returns a normal-looking response. Verify the negative case, that the wrong role, wrong tenant, or wrong owner is actually rejected, not just that the right one succeeds.

**Infrastructure.** Deployment topology, service boundaries, environment configuration, secrets and credentials, external service contracts, networking and access rules, resource limits. Also: does this change require a deploy order, and does anything enforce it.

**Background jobs.** Workers, cron, queues, webhooks, scheduled tasks, retry handlers, anything running outside the request that triggered the change. Silent-failure prone in a specific way: a job that stops running rarely raises anything. Ask what happens to work already queued under the old contract.

**Tests.** What coverage exists over the reach identified in steps 1 and 2, not what coverage exists over the changed module, which is a different and easier question. Name the gaps: reach with no test is the part of the map most worth acting on, since it's where a mistake goes undetected longest.

**Backwards compatibility.** What breaks for consumers on the old shape, and for how long. Whether old and new can coexist, or whether this is a hard cutover. Whether there's a deprecation path. If the answer is "everything deploys together," that's a real answer. Verify it rather than assuming it.
