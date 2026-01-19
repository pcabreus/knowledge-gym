---
id: postgresql-bloat
title: "PostgreSQL Bloat"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [database, postgresql, performance]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# PostgreSQL Bloat

## TL;DR (BLUF)
- Bloat is wasted space from dead tuples and index growth.
- Use vacuum and good update patterns to control it.
- Trade-off: aggressive cleanup increases write overhead.

## Definition
**What it is:** Extra table/index space that isn’t actively used but not reclaimed.
**Key terms:** dead tuples, vacuum, fillfactor.

## Why it matters
- Bloat increases I/O and slows queries.
- It can silently grow storage costs.

## Scope & Non-goals
**In scope:** bloat causes and mitigations.
**Out of scope / NOT solved by this:** storage layer compression.

## Mental model / Intuition
- Think of bloat as “empty seats” left behind by row updates.

## Decision rules (When to use / When not to use)
### Use it when
- You see table size growing faster than data.
### Avoid it when
- You haven’t validated dead tuples or table stats.

## How I would use it (practical)
- **Context:** table grows despite stable row count.
- **Steps:** measure dead tuples → tune autovacuum → consider VACUUM FULL.
- **What success looks like:** stable table/index size.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** lower I/O and better cache utilization.
- **Cons / Risks:** cleanup overhead.
### Alternatives
- **VACUUM FULL:** heavy but compacting.
- **How to choose:** use regular vacuum; reserve full vacuum for extreme bloat.

## Failure modes & Pitfalls
- Long transactions preventing cleanup.
- Misreading table size growth.

## Observability (How to detect issues)
- **Metrics:** dead tuples, table size, cache hit ratio.
- **Logs:** autovacuum logs.
- **Alerts:** increasing bloat indicators.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Track dead tuples
  - [ ] Tune autovacuum
  - [ ] Review update patterns

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Ignoring bloat until disks fill.
  - **Why it’s bad:** performance degrades gradually.
  - **Better approach:** monitor and tune early.

## Interview readiness
### “Explain it like I’m teaching”
- Bloat is wasted space from old row versions in Postgres. Vacuum cleans it, but if it lags, tables and indexes grow and queries slow down.

### Trap questions (with answers)
1) **Q:** Does vacuum always shrink table files?
   - **A:** no; it mainly frees space for reuse.
2) **Q:** Is bloat only a table problem?
   - **A:** no; indexes can bloat too.
3) **Q:** Can long transactions cause bloat?
   - **A:** yes; they prevent cleanup of old versions.

### Quick self-check (5 items)
- [ ] I can define bloat.
- [ ] I can explain how it happens.
- [ ] I can name a mitigation.
- [ ] I can describe a signal.
- [ ] I can relate it to MVCC.

## Links (NO duplication)
### Prerequisites
- [PostgreSQL MVCC](postgresql-mvcc.md)

### Related topics
- [PostgreSQL vacuum and autovacuum](postgresql-vacuum-autovacuum.md)

### Compare with
- [Storage compaction](storage-compaction.md)
