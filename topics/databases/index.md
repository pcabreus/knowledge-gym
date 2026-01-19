---
id: index
title: "Index (Database Index)"
type: concept
status: learning
importance: 85
difficulty: medium
tags: [database, performance]
canonical: true
aliases: ["db-index"]
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Index (Database Index)

## TL;DR (BLUF)
- An index is a data structure that speeds up reads at the cost of writes and storage.
- Use it for selective filters, joins, and ordering on critical queries.
- Trade-off: too many or wrong indexes hurt write performance and maintenance.

## Definition
**What it is:** A structure that lets the database locate rows faster than a full scan.
**Key terms:** selectivity, query planner, B-Tree, composite index.

## Why it matters
- It’s often the difference between fast queries and timeouts on large datasets.
- Bad indexing decisions can silently degrade write throughput.

## Scope & Non-goals
**In scope:** improving read performance for known access patterns.
**Out of scope / NOT solved by this:** fixing poorly designed queries or missing pagination limits.

## Mental model / Intuition
- Like a book index: you trade extra pages to find chapters quickly.
- You pay on writes to keep the index current.

## Decision rules (When to use / When not to use)
### Use it when
- A query is slow and selective (WHERE / JOIN / ORDER BY) and appears frequently.
- EXPLAIN shows costly scans.
### Avoid it when
- The column has low selectivity and no supporting strategy.
- The table is write-heavy and the index isn’t critical.

## How I would use it (practical)
- **Context:** A high-traffic API with slow read endpoints.
- **Steps:** identify slow queries → run EXPLAIN → add the smallest index that helps → measure → monitor.
- **What success looks like:** lower p95 latency with minimal write impact.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** reduces read latency.
- **Cons / Risks:** worsens writes, increases storage, can add contention.
### Alternatives
- **Materialized views:** for heavy read workloads.
- **Denormalization:** when joins are the bottleneck.
- **App-level caching:** when data freshness allows it.
- **How to choose:** if reads are critical and selective, indexes are usually the first step.

## Failure modes & Pitfalls
- Index not used because the query doesn’t match the index order.
- Index bloat from frequent updates.
- Redundant indexes increasing write cost without benefit.

## Observability (How to detect issues)
- **Metrics:** p95 query latency, index usage ratio, bloat, write latency.
- **Logs:** slow query logs, planner decisions.
- **Traces:** DB spans showing full table scans.
- **Alerts:** sustained p95 spikes or increasing write latency after index changes.

## Implementation notes (if applicable)
- **Checklist**
   - [ ] Identify critical queries
   - [ ] Review selectivity
   - [ ] Validate with EXPLAIN
   - [ ] Measure impact on writes
- **Performance notes**
   - Prefer the smallest index that satisfies the query.
- **Operational notes**
   - Periodically review unused indexes.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** “Index all the things.”
   - **Why it’s bad:** high write cost with little gain.
   - **Better approach:** index only for known query patterns.
- **Anti-pattern:** Ignoring index order in composite indexes.
   - **Why it’s bad:** the planner can’t use the index effectively.
   - **Better approach:** order columns to match WHERE/JOIN/ORDER BY.

## Interview readiness
### “Explain it like I’m teaching”
- An index is like a lookup table that speeds up reads by trading extra storage and write overhead. Use it for frequent, selective queries; avoid it for write-heavy tables or low-selectivity columns.

### Trap questions (with answers)
1) **Q:** Why do too many indexes hurt the system?
    - **A:** they penalize INSERT/UPDATE/DELETE because each index must be updated.
2) **Q:** Is an index always used?
    - **A:** no; the planner may skip it if the estimated cost doesn’t pay off.
3) **Q:** Does an index fix any slow query?
    - **A:** no; if the filter doesn’t reduce much or the query is poorly designed, the index won’t help.

### Quick self-check (5 items)
- [ ] I can define an index precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [PostgreSQL](postgresql.md)

### Related topics
- [JSONB](jsonb.md)
- [GIN index](gin-index.md)

### Compare with
- (TODO) B-Tree vs GIN vs GiST
