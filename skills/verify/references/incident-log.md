# Incident Log

Add an entry every time something passed typecheck + tests and still shipped broken, or would have shipped broken if not caught late. This file is what makes `verify` yours instead of a generic checklist: the point is that it gets more specific to your own projects over time.

Format:

```
## YYYY-MM-DD: short title

**What looked done:** what the static checks / test suite reported
**What was actually broken:** the real behavior gap
**Where:** repo / package / module
**New check this adds:** what this skill should now check for that it didn't before
**Codified as:** new check in this skill / new impact category / lint rule / nothing (one-time fluke, say why)
```

---

(No entries yet. Add the first one the next time this happens, rather than trying to backfill from memory. A live log built from real incidents as they occur will be more accurate than a reconstructed one.)