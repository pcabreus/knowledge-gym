---
id: selectivity
title: "Selectivity (Database Queries)"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [database, performance]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Selectivity (Database Queries)

## TL;DR (BLUF)
- Selectivity measures how well a predicate filters rows.
- Use it to decide if an index will actually be helpful.
- Trade-off: high-selectivity indexes help reads but add write cost.

## Definition
**What it is:** The fraction of rows that satisfy a predicate; higher selectivity means fewer rows match.
**Key terms:** predicate, cardinality, selectivity, query planner.

## Why it matters
- The query planner chooses index usage based on selectivity estimates.
- Low selectivity often means indexes are ignored or ineffective.

## Scope & Non-goals
**In scope:** deciding index usefulness for filters and joins.
**Out of scope / NOT solved by this:** fixing poor query logic or data modeling.

## Mental model / Intuition
- A highly selective filter is like a needle in a haystack; indexes shine.

## Decision rules (When to use / When not to use)
### Use it when
- Choosing indexes for predicates and joins.
- Evaluating whether a query can benefit from an index.
### Avoid it when
- You need to optimize queries without understanding actual data distribution.

## How I would use it (practical)
- **Context:** Slow query with a WHERE filter.
- **Steps:** check distribution → estimate selectivity → decide index or query rewrite.
- **What success looks like:** planner uses the index and latency drops.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** better index decisions, fewer wasted indexes.
- **Cons / Risks:** estimates can be wrong if stats are stale.
### Alternatives
- **EXPLAIN:** validate planner decisions.
- **How to choose:** use selectivity to guide index design, then validate with EXPLAIN.

## Failure modes & Pitfalls
- Stale statistics causing wrong selectivity estimates.
- Assuming selectivity is constant across time.

## Observability (How to detect issues)
- **Metrics:** index usage ratio, query latency.
- **Logs:** slow query logs.
- **Alerts:** rising latency after data distribution shifts.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Update statistics (ANALYZE)
  - [ ] Validate with EXPLAIN

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Indexing low-selectivity columns without strategy.
  - **Why it’s bad:** index is often ignored; writes get slower.
  - **Better approach:** index composite keys or rethink the predicate.

## Interview readiness
### “Explain it like I’m teaching”
- Selectivity tells you how many rows a filter matches. If it matches too many rows, the planner may skip the index, so understanding selectivity is key to good indexing.

### Trap questions (with answers)
1) **Q:** Do indexes always help?
   - **A:** no; low-selectivity predicates often don’t benefit.
2) **Q:** Can selectivity change over time?
   - **A:** yes; data distribution shifts can change planner choices.
3) **Q:** Is selectivity only about equality filters?
   - **A:** no; it applies to ranges and joins too.

### Quick self-check (5 items)
- [ ] I can define selectivity precisely.
- [ ] I can explain why it matters for indexing.
- [ ] I can name 2 trade-offs.
- [ ] I can give a concrete example.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Index](index.md)

### Related topics
- [EXPLAIN](explain.md)

### Compare with
- [EXPLAIN](explain.md) — planner output vs data distribution.
