# Boundary Patterns — scope

Two things: how to draw the required/related line when it's genuinely unclear, and the recurring ways scope moves without anyone deciding to move it.

---

## Drawing the required/related line

**The removal test.** Take the item out. Is the stated outcome still true? If yes, it's related. If no, it's required.

Apply it to the outcome as the requester stated it, not as you'd have stated it. This is where step 1 pays off: an outcome quietly rephrased into something more specific pulls work across the line with it, and the rephrasing is what did it, not any decision anyone made.

**Three cases the removal test doesn't settle:**

*Required for correctness, not for the feature.* A permission check the feature doesn't need to function but does need in order to be safe to ship. Required — "works" and "shippable" aren't different outcomes when the gap is a security or data-integrity hole. Say why it's required rather than letting it look like scope creep.

*Required by a dependency, not the request.* The requested change needs a library upgrade, which needs a config migration. Required, but list the chain explicitly. A dependency that pulls in three unrelated items is a signal worth surfacing before starting, not after.

*Required to avoid leaving things broken.* Changing a shared function means updating its callers. The callers weren't in the request; leaving them broken isn't an option. Required — but if the chain is long enough that it doubles the work, that's a scope conversation to have now, not a discovery to make at hour six.

---

## How scope silently expands

**While you're in there.** The most common one. The file is open, the problem is visible, the fix is small. Each instance is individually reasonable, which is exactly why it accumulates without a decision ever being made. Route it to `related`. The cost isn't the fix — it's that the change now contains two things, and reverting one reverts the other.

**Fixing the thing you had to understand.** Understanding the code to make the change reveals something wrong with it. Real knowledge, genuinely worth acting on, and still a separate change. Note it, route it to related, do it next.

**Generalizing for the second caller that doesn't exist.** The change would be more elegant as a general mechanism. Sometimes true. It's still additional work justified by a hypothetical, and it belongs in `related` where someone can weigh it, not folded in as though the request implied it.

**Scope inherited from the fix.** The chosen approach requires groundwork the request never mentioned — a refactor to make the change clean, a new abstraction to hold it. Legitimate, and it needs to be visible: the requester asked for an outcome, not for an approach that costs three times as much. If the groundwork dominates, that's a decision for them.

**Delegation drift.** An agent given a vague task fills the gap with something plausible and builds it. It doesn't stop to ask, and the invented scope is usually larger than the real one and always more confident. This is the case the whole skill is for — a delegated task with no stated boundary gets one anyway, just not from you.

---

## How scope silently shrinks

Less discussed than creep, and more expensive, because the gap surfaces at review instead of during the work.

**The obvious part nobody said.** The requester assumed something was included because it was obvious to them. It wasn't in the sentence. This is what step 4 catches — an explicit exclusion gives them the chance to say "wait, I assumed that was in."

**Deferred and forgotten.** "We'll do that after" with no record. Indistinguishable from never planning to do it. If it's deferred, it goes in `related` where it stays visible.

**Blocked and quietly dropped.** A required item turns out to be hard, so it goes unfinished while everything else lands. The work looks done. Re-scope out loud instead: say what's blocked and what shipping without it means.

**Narrowed by the approach.** The chosen implementation can only handle part of the request, and the request quietly becomes what the approach handles. Notice when the solution is defining the problem.

---

## When the request is genuinely ambiguous

Don't resolve it by picking the reading you'd rather build.

Record the ambiguity as an open question, and where you must proceed, state the assumption in the scope statement so it can be corrected cheaply. "Reading this as X, not Y — say if that's wrong" costs one line now, or a rebuild later.

Two readings that produce materially different work is worth a question before starting. Two readings that converge on nearly the same work is worth an assumption and no interruption. The difference is whether being wrong is expensive, not whether you're uncertain.
