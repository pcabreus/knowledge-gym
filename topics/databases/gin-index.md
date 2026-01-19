---
id: gin-index
title: "GIN index (PostgreSQL)"
type: concept
status: learning
importance: 60
difficulty: medium
tags: [database, postgresql, indexing]
canonical: true
aliases: ["gin"]
created_at: 2026-01-19
updated_at: 2026-01-19
---

# GIN index (PostgreSQL)

## TL;DR (BLUF)
- GIN is an inverted index optimized for searching inside collections (arrays/JSONB).
- Use it when you query JSONB keys/paths or array contents.
- Trade-off: higher write and maintenance cost; operators must match the index.

## Definition
**What it is:** A Generalized Inverted Index that maps elements to rows, enabling fast containment and membership queries.
**Key terms:** inverted index, operators, JSONB, array containment.

## Why it matters
- It unlocks efficient queries over internal content where B-Tree is ineffective.
- It’s the standard index choice for JSONB containment queries.

## Scope & Non-goals
**In scope:** containment/membership queries on JSONB and arrays.
**Out of scope / NOT solved by this:** scalar equality/range queries and poor data modeling.

## Mental model / Intuition
- Imagine a tag-to-document map: each element points to all rows containing it.
- Great for “contains X” queries; not ideal for simple equality on scalar columns.

## Decision rules (When to use / When not to use)
### Use it when
- You query JSONB by keys/paths with operators that GIN supports.
- Your access pattern is “contains X inside the structure.”
### Avoid it when
- You rarely query internal content.
- Your workload is extremely write-heavy and index cost is too high.

## How I would use it (practical)
- **Context:** A table with JSONB attributes queried by key/value.
- **Steps:** identify JSONB access patterns → create GIN index → validate with EXPLAIN → monitor write impact.
- **What success looks like:** faster JSONB queries without unacceptable write degradation.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** fast internal searches for JSONB/arrays.
- **Cons / Risks:** write amplification, maintenance cost, operator sensitivity.
### Alternatives
- **B-Tree:** best for equality/range on scalar columns.
- **GiST:** useful for other data types (e.g., geospatial).
- **How to choose:** pick GIN for containment queries; otherwise consider B-Tree/GiST.

## Failure modes & Pitfalls
- Index not used due to operator mismatch.
- Significant write slowdown from heavy updates.
- Index bloat from frequent modifications.

## Observability (How to detect issues)
- **Metrics:** query latency for JSONB filters, write latency, index size growth.
- **Logs:** slow query logs for JSONB operators.
- **Traces:** DB spans showing JSONB queries as hotspots.
- **Alerts:** sustained write latency increase after index creation.

## Implementation notes (if applicable)
- **Checklist**
   - [ ] Confirm JSONB access patterns and operators
   - [ ] Create GIN index and validate with EXPLAIN
   - [ ] Measure write impact
- **Performance notes**
   - Limit JSONB size and avoid unnecessary updates.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Creating GIN “just in case.”
   - **Why it’s bad:** write overhead without query benefit.
   - **Better approach:** add it only for proven access patterns.
- **Anti-pattern:** Using JSONB for everything and relying on GIN to fix it.
   - **Why it’s bad:** it masks poor modeling and hurts writes.
   - **Better approach:** model stable fields as columns and use JSONB for truly variable data.

## Interview readiness
### “Explain it like I’m teaching”
- GIN is an inverted index in PostgreSQL that’s great for searching inside JSONB or arrays. It’s fast for containment queries but more expensive on writes and only helps when operators match the index.

### Trap questions (with answers)
1) **Q:** Is GIN always better than B-Tree?
    - **A:** no; it depends on access type. B-Tree is ideal for equality/ranges on scalar values.
2) **Q:** If I have GIN, do I no longer need to model?
    - **A:** false; indexes help, but they don’t fix a poor model/queries.
3) **Q:** Does a GIN not affect writes?
    - **A:** it does affect writes: INSERT/UPDATE must update index structures.

### Quick self-check (5 items)
- [ ] I can define GIN precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete usage example.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Index](index.md)
- [JSONB](jsonb.md)
- [PostgreSQL](postgresql.md)

### Related topics
- [B-Tree index](btree-index.md)
- [GiST](gist-index.md)
- [Selectivity](selectivity.md)
- [EXPLAIN](explain.md)
