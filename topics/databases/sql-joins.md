---
id: sql-joins
title: "SQL Joins (Inner/Left)"
type: concept
status: learning
importance: 65
difficulty: easy
tags: [database, sql]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# SQL Joins (Inner/Left)

## TL;DR (BLUF)
- Joins combine rows from two tables based on a predicate.
- Use INNER for matching rows only; LEFT to keep all rows from the left table.
- Trade-off: joins can be expensive without indexes and good predicates.

## Definition
**What it is:** A relational operation that merges rows from tables using a join condition.
**Key terms:** INNER JOIN, LEFT JOIN, join predicate, cardinality.

## Why it matters
- Joins are core to relational modeling and query expressiveness.
- Misused joins cause incorrect results and performance issues.

## Scope & Non-goals
**In scope:** INNER and LEFT joins, correctness and performance basics.
**Out of scope / NOT solved by this:** advanced join algorithms and query planner internals.

## Mental model / Intuition
- Think of joining as “matching rows” based on a key.

## Decision rules (When to use / When not to use)
### Use it when
- You need related data from multiple tables.
- You have clear keys and join predicates.
### Avoid it when
- A single table can answer the query with simpler filters.
- The join predicate is not selective and causes huge result sets.

## How I would use it (practical)
- **Context:** Users and orders tables.
- **Steps:** choose INNER vs LEFT → join on indexed keys → validate row counts.
- **What success looks like:** correct results with stable performance.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** normalized data and flexible queries.
- **Cons / Risks:** performance cost on large joins.
### Alternatives
- **Denormalization:** pre-join data when reads dominate.
- **How to choose:** use joins when correctness matters and data is normalized.

## Failure modes & Pitfalls
- Accidental cartesian products from missing predicates.
- LEFT join turning into INNER due to WHERE filters on right table.

## Observability (How to detect issues)
- **Metrics:** query latency, rows scanned.
- **Logs:** slow query logs.
- **Alerts:** sudden spikes in query runtime.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Ensure join keys are indexed
  - [ ] Verify expected row counts

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Joining without indexed keys.
  - **Why it’s bad:** full scans and slow queries.
  - **Better approach:** index join columns.

## Interview readiness
### “Explain it like I’m teaching”
- SQL joins combine data from related tables. INNER returns only matches; LEFT keeps all left rows even if there’s no match. Correct predicates and indexes keep joins fast.

### Trap questions (with answers)
1) **Q:** Does LEFT JOIN always return more rows than INNER?
   - **A:** not necessarily; it can be the same if all rows match.
2) **Q:** Can a LEFT JOIN turn into an INNER JOIN accidentally?
   - **A:** yes, if you filter on right-table columns in WHERE.
3) **Q:** Are joins always slow?
   - **A:** no; with proper indexes and selectivity they can be fast.

### Quick self-check (5 items)
- [ ] I can define INNER vs LEFT joins.
- [ ] I can state when to use each.
- [ ] I can explain a performance trade-off.
- [ ] I can describe a common pitfall.
- [ ] I can name a detection signal.

## Links (NO duplication)
### Prerequisites
- [SQL foundations](sql-foundations.md)

### Related topics
- [Index](index.md)

### Compare with
- [Normalization](normalization.md) — normalized models rely on joins.
