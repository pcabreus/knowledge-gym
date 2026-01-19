---
id: explain
title: "EXPLAIN (PostgreSQL)"
type: concept
status: learning
importance: 60
difficulty: medium
tags: [database, postgresql, performance]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# EXPLAIN (PostgreSQL)

## TL;DR (BLUF)
- EXPLAIN shows the query plan and cost estimates chosen by the planner.
- Use it to validate indexes and query shape before optimizing.
- Trade-off: plans are estimates; they can be wrong if stats are stale.

## Definition
**What it is:** A command that outputs how PostgreSQL plans to execute a query.
**Key terms:** query plan, cost estimate, planner, ANALYZE.

## Why it matters
- It helps you understand why a query is slow.
- It prevents guesswork when adding or removing indexes.

## Scope & Non-goals
**In scope:** diagnosing query plans and validating indexes.
**Out of scope / NOT solved by this:** fixing logic errors in SQL.

## Mental model / Intuition
- Think of EXPLAIN as a “map” of how the DB will walk through data.

## Decision rules (When to use / When not to use)
### Use it when
- You’re adding indexes or optimizing slow queries.
- You want to verify planner choices for critical queries.
### Avoid it when
- You need execution time without running the query (use EXPLAIN ANALYZE carefully).

## How I would use it (practical)
- **Context:** A slow endpoint query.
- **Steps:** run EXPLAIN → check seq scan vs index scan → adjust indexes → rerun.
- **What success looks like:** planner uses expected indexes and cost drops.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** visibility into planner decisions.
- **Cons / Risks:** can mislead if stats are stale or data distribution changed.
### Alternatives
- **EXPLAIN ANALYZE:** actual execution time, but runs the query.
- **How to choose:** start with EXPLAIN, validate with EXPLAIN ANALYZE when safe.

## Failure modes & Pitfalls
- Misreading costs as actual time.
- Ignoring row estimates and join order.

## Observability (How to detect issues)
- **Metrics:** slow query frequency, planner stats freshness.
- **Logs:** slow query logs with plan capture if available.
- **Alerts:** high latency after schema or data changes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Ensure stats are fresh (ANALYZE)
  - [ ] Compare row estimates vs actual with EXPLAIN ANALYZE

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Adding indexes without checking EXPLAIN.
  - **Why it’s bad:** you may add indexes the planner won’t use.
  - **Better approach:** use EXPLAIN to guide index design.

## Interview readiness
### “Explain it like I’m teaching”
- EXPLAIN shows how PostgreSQL plans to run a query and whether it will use indexes. It’s the first tool you should use before making performance changes.

### Trap questions (with answers)
1) **Q:** Does EXPLAIN show actual runtime?
   - **A:** no; that’s EXPLAIN ANALYZE.
2) **Q:** Can EXPLAIN be wrong?
   - **A:** yes; if statistics are stale, the plan may be misleading.
3) **Q:** Should you always trust the planner?
   - **A:** mostly, but validate when results look off.

### Quick self-check (5 items)
- [ ] I can define EXPLAIN precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete optimization example.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Index](index.md)

### Related topics
- [Selectivity](selectivity.md)

### Compare with
- [Selectivity](selectivity.md) — planner estimates vs data distribution.
