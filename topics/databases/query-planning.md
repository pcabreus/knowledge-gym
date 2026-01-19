---
id: query-planning
title: "Query Planning"
type: concept
status: learning
importance: 60
difficulty: medium
tags: [database, performance]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Query Planning

## TL;DR (BLUF)
- The query planner chooses how to execute a query based on cost estimates.
- Use planner tools to understand index usage and join order.
- Trade-off: plans are estimates and can be wrong if stats are stale.

## Definition
**What it is:** The database component that decides execution strategies (scan types, join order, indexes).
**Key terms:** cost estimate, statistics, join order, scan type.

## Why it matters
- Planner choices often determine query performance.
- Wrong estimates lead to slow queries and misused indexes.

## Scope & Non-goals
**In scope:** planning concepts and how to validate them.
**Out of scope / NOT solved by this:** specific optimizer algorithms per DB.

## Mental model / Intuition
- Think of it as a route planner choosing the cheapest path.

## Decision rules (When to use / When not to use)
### Use it when
- Diagnosing slow queries or adding indexes.
### Avoid it when
- You only need to validate runtime (use EXPLAIN ANALYZE carefully).

## How I would use it (practical)
- **Context:** Query latency spike after data growth.
- **Steps:** run EXPLAIN → check scan types → verify join order → adjust indexes/stats.
- **What success looks like:** planner picks efficient scans and join order.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** visibility into performance decisions.
- **Cons / Risks:** cost estimates can be wrong.
### Alternatives
- **EXPLAIN ANALYZE:** actual timings, but executes the query.
- **How to choose:** start with EXPLAIN, validate with ANALYZE if safe.

## Failure modes & Pitfalls
- Stale statistics causing bad plans.
- Misreading cost as actual time.

## Observability (How to detect issues)
- **Metrics:** slow query frequency, plan changes over time.
- **Logs:** slow query logs.
- **Alerts:** sudden latency spikes after data distribution changes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Keep stats fresh (ANALYZE)
  - [ ] Compare estimates vs actuals

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Adding indexes without checking plans.
  - **Why it’s bad:** planner may ignore them.
  - **Better approach:** validate with EXPLAIN.

## Interview readiness
### “Explain it like I’m teaching”
- Query planning is the DB deciding the cheapest way to run a query. It uses statistics to pick indexes and join order, and you validate it with tools like EXPLAIN.

### Trap questions (with answers)
1) **Q:** Is the planner always correct?
   - **A:** no; bad stats or skew can mislead it.
2) **Q:** Does EXPLAIN show actual runtime?
   - **A:** no; EXPLAIN ANALYZE does.
3) **Q:** Can indexes be ignored?
   - **A:** yes, if the planner estimates they won’t help.

### Quick self-check (5 items)
- [ ] I can define query planning.
- [ ] I can explain why stats matter.
- [ ] I can name a failure mode.
- [ ] I can describe a validation step.
- [ ] I can compare EXPLAIN vs EXPLAIN ANALYZE.

## Links (NO duplication)
### Prerequisites
- [EXPLAIN](explain.md)

### Related topics
- [Selectivity](selectivity.md)

### Compare with
- [EXPLAIN](explain.md) — planner output vs execution.
