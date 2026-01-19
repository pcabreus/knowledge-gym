---
id: btree-index
title: "B-Tree Index"
type: concept
status: learning
importance: 60
difficulty: medium
tags: [database, indexing]
canonical: true
aliases: ["b-tree"]
created_at: 2026-01-19
updated_at: 2026-01-19
---

# B-Tree Index

## TL;DR (BLUF)
- Default index type for equality and range queries on scalar columns.
- Use it for ORDER BY, BETWEEN, and exact matches.
- Trade-off: not ideal for containment queries in arrays/JSON.

## Definition
**What it is:** A balanced tree index optimized for ordered data and range scans.
**Key terms:** equality, range scan, ordering, selectivity.

## Why it matters
- It’s the most common index type and powers fast range/equality queries.
- Misusing it for containment patterns leads to poor performance.

## Scope & Non-goals
**In scope:** scalar equality/range predicates, ORDER BY, prefix matches.
**Out of scope / NOT solved by this:** containment queries inside JSON/arrays.

## Mental model / Intuition
- Think of a sorted phone book: binary search and range scans are fast.

## Decision rules (When to use / When not to use)
### Use it when
- You filter by equality or ranges on scalar columns.
- You need fast ORDER BY or LIMIT + ORDER BY.
### Avoid it when
- You query inside arrays or JSON documents.
- You need geospatial or full-text style queries.

## How I would use it (practical)
- **Context:** Filtering by `created_at` or `user_id` with ordering.
- **Steps:** add B-Tree index → validate with EXPLAIN → monitor usage.
- **What success looks like:** low latency and index usage in query plans.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** great for ordered data and range queries.
- **Cons / Risks:** not helpful for containment queries.
### Alternatives
- **GIN index:** for JSONB/array containment.
- **GiST:** for specialized types like geospatial.
- **How to choose:** use B-Tree for scalar equality/ranges; switch for specialized operators.

## Failure modes & Pitfalls
- Index not used due to low selectivity or function wrapping.
- Composite index order mismatch with queries.

## Observability (How to detect issues)
- **Metrics:** index usage, query latency on range filters.
- **Logs:** slow query logs.
- **Alerts:** sustained latency on ordered queries.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Ensure predicates match index order
  - [ ] Validate with EXPLAIN

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using B-Tree for JSON containment.
  - **Why it’s bad:** it won’t use the index efficiently.
  - **Better approach:** use [GIN index](gin-index.md).

## Interview readiness
### “Explain it like I’m teaching”
- A B-Tree index is the default index for ordered data. It’s ideal for equality and range queries, but it doesn’t help with containment queries inside JSON or arrays.

### Trap questions (with answers)
1) **Q:** Is B-Tree always the best index type?
	- **A:** no; it depends on operators and access patterns.
2) **Q:** Does B-Tree speed up JSONB containment?
	- **A:** no; use GIN for containment queries.
3) **Q:** Can index order matter?
	- **A:** yes; composite indexes are order-sensitive.

### Quick self-check (5 items)
- [ ] I can define B-Tree indexes precisely.
- [ ] I can state when to use them and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a range-query example.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Index](index.md)

### Related topics
- [GIN index](gin-index.md)

### Compare with
- [GIN index](gin-index.md) — containment queries vs ordered scans.
- [GiST](gist-index.md) — specialized types vs ordered scalars.
