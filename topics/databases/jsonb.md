---
id: jsonb
title: "JSONB (PostgreSQL)"
type: concept
status: learning
importance: 70
difficulty: medium
tags: [database, postgresql]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# JSONB (PostgreSQL)

## TL;DR (BLUF)
- JSONB stores JSON in a binary format with operators and indexing support in PostgreSQL.
- Use it for semi-structured or evolving fields that still need transactions and joins.
- Trade-off: weaker structure guarantees and potential performance issues if abused.

## Definition
**What it is:** A PostgreSQL data type that stores JSON in a binary format optimized for querying and indexing.
**Key terms:** JSONB, operators, GIN index.

## Why it matters
- It provides schema flexibility without leaving a relational database.
- It enables search over JSON keys/paths with proper indexing.

## Scope & Non-goals
**In scope:** semi-structured attributes, flexible metadata, custom fields.
**Out of scope / NOT solved by this:** strict relational modeling for stable schemas and guaranteed JSON integrity without extra validation.

## Mental model / Intuition
- Think of JSONB as a “flexible envelope” attached to a relational row.
- It’s powerful for variability, but you must decide what belongs in columns vs JSONB.

## Decision rules (When to use / When not to use)
### Use it when
- You have evolving attributes or sparse fields.
- You still need SQL joins and ACID transactions.
### Avoid it when
- The schema is stable and frequently queried.
- You require strong structure enforcement without additional validation.

## How I would use it (practical)
- **Context:** Feature flags or custom fields per tenant.
- **Steps:** model stable fields as columns → store variable fields in JSONB → add [GIN index](gin-index.md) for access paths → validate JSON in app.
- **What success looks like:** flexible schema with predictable query latency and maintainable data integrity.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** flexibility and expressive queries.
- **Cons / Risks:** schema-less mess, validation burden, potential performance cost.
### Alternatives
- **Normal columns:** better integrity and clarity for stable fields.
- **EAV table:** for complex attribute queries (with trade-offs).
- **How to choose:** use JSONB only for truly variable parts of the model.

## Failure modes & Pitfalls
- JSONB grows unbounded, causing bloat and slow updates.
- Queries become slow due to missing or wrong indexes.
- Inconsistent shape across rows leads to application logic complexity.

## Observability (How to detect issues)
- **Metrics:** query latency on JSONB filters, index usage, bloat.
- **Logs:** slow query logs for JSON path queries.
- **Traces:** DB spans showing JSONB operators as hotspots.
- **Alerts:** increasing latency for JSONB-heavy endpoints.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Identify variable fields vs stable columns
  - [ ] Add [GIN index](gin-index.md) for access paths
  - [ ] Validate JSON shape at the app layer
- **Performance notes**
  - Keep JSONB documents small and focused.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Storing full entities in JSONB.
  - **Why it’s bad:** you lose relational constraints and efficient indexing.
  - **Better approach:** keep core fields as columns and use JSONB for true flex fields.

## Interview readiness
### “Explain it like I’m teaching”
- JSONB is PostgreSQL’s binary JSON type. It’s great for flexible or evolving attributes while still using SQL, but it shifts validation to the application and can hurt performance if used for everything.

### Trap questions (with answers)
1) **Q:** Is JSONB always faster than JSON?
   - **A:** it depends; JSONB enables operations/indexing, but adds storage/transform cost.
2) **Q:** Does JSONB eliminate migrations?
   - **A:** it reduces some, but shifts the problem to validation/consistency at the app level.
3) **Q:** Can JSONB be indexed?
   - **A:** yes, with index types like [GIN index](gin-index.md) depending on access patterns.

### Quick self-check (5 items)
- [ ] I can define JSONB precisely in 2–3 sentences.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete use case.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [PostgreSQL](postgresql.md)
- [Index](index.md)

### Related topics
- [GIN index](gin-index.md)

### Compare with
- (TODO) MongoDB documents
