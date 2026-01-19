---
id: postgresql-stats-and-analyze
title: "PostgreSQL Statistics & ANALYZE"
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

# PostgreSQL Statistics & ANALYZE

## TL;DR (BLUF)
- The planner relies on table statistics to estimate costs and choose plans.
- Use ANALYZE to keep stats fresh after big data changes.
- Trade-off: collecting stats adds overhead.

## Definition
**What it is:** Statistics about data distribution used by the planner, updated by ANALYZE.
**Key terms:** statistics, ANALYZE, histograms.

## Why it matters
- Stale stats lead to bad plans and slow queries.
- Good stats improve index selection and join order.

## Scope & Non-goals
**In scope:** stats freshness and its impact on planning.
**Out of scope / NOT solved by this:** query rewrite best practices.

## Mental model / Intuition
- Think of stats as the planner’s map of your data.

## Decision rules (When to use / When not to use)
### Use it when
- You load or delete large amounts of data.
### Avoid it when
- You can rely on autovacuum/analyze for small changes.

## How I would use it (practical)
- **Context:** Bulk load followed by slow queries.
- **Steps:** run ANALYZE → re-check EXPLAIN → monitor plan changes.
- **What success looks like:** planner chooses efficient scans.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** better plans and performance.
- **Cons / Risks:** overhead during stats collection.
### Alternatives
- **Autovacuum analyze:** automatic stats updates.
- **How to choose:** manual ANALYZE after large data shifts.

## Failure modes & Pitfalls
- Assuming stats are updated immediately after bulk loads.
- Misreading planner estimates without checking stats.

## Observability (How to detect issues)
- **Metrics:** plan changes, query latency after bulk loads.
- **Logs:** autovacuum/analyze logs.
- **Alerts:** latency spikes after large data changes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Run ANALYZE after bulk changes
  - [ ] Monitor plan stability

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Ignoring stats after large deletes.
  - **Why it’s bad:** planner chooses poor plans.
  - **Better approach:** analyze after major data shifts.

## Interview readiness
### “Explain it like I’m teaching”
- PostgreSQL uses statistics to decide how to run queries. After big data changes, run ANALYZE so the planner has accurate info; otherwise, it may pick bad plans.

### Trap questions (with answers)
1) **Q:** Does ANALYZE rewrite data on disk?
   - **A:** no; it only updates statistics.
2) **Q:** Are stats always perfectly accurate?
   - **A:** no; they’re estimates.
3) **Q:** Is autovacuum enough after a bulk load?
   - **A:** not always; manual ANALYZE can help.

### Quick self-check (5 items)
- [ ] I can explain why stats matter.
- [ ] I can name when to run ANALYZE.
- [ ] I can describe a trade-off.
- [ ] I can identify a failure mode.
- [ ] I can relate stats to query planning.

## Links (NO duplication)
### Prerequisites
- [Query planning](query-planning.md)
- [EXPLAIN](explain.md)

### Related topics
- [Selectivity](selectivity.md)

### Compare with
- [Database statistics](database-statistics.md)
