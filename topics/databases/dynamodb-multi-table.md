---
id: dynamodb-multi-table
title: "DynamoDB Multi-Table Design"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [database, dynamodb, nosql]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# DynamoDB Multi-Table Design

## TL;DR (BLUF)
- Multi-table design models each entity or access pattern in its own table.
- Use it when access patterns are simple or teams need isolation.
- Trade-off: more tables and operational overhead.

## Definition
**What it is:** A DynamoDB modeling approach that uses multiple tables for different entities or access patterns.
**Key terms:** table-per-entity, access pattern, isolation.

## Why it matters
- It’s simpler to reason about but can limit cross-entity queries.
- It can reduce complexity compared to single-table design.

## Scope & Non-goals
**In scope:** when to split into multiple tables.
**Out of scope / NOT solved by this:** ad-hoc queries across entities.

## Mental model / Intuition
- Think “one table per domain concept.”

## Decision rules (When to use / When not to use)
### Use it when
- Access patterns are straightforward and do not need cross-entity joins.
- You want team or domain isolation.
### Avoid it when
- You need multiple access patterns across entities in one table.

## How I would use it (practical)
- **Context:** Separate tables for users and orders.
- **Steps:** define access patterns per table → design keys → keep GSIs minimal.
- **What success looks like:** simple queries with low operational overhead.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** simpler design and debugging.
- **Cons / Risks:** more tables, less flexibility.
### Alternatives
- **Single-table design:** more powerful but complex.
- **How to choose:** use multi-table when simplicity and isolation are priorities.

## Failure modes & Pitfalls
- Duplicated data across tables for access patterns.

## Observability (How to detect issues)
- **Metrics:** per-table throttles, latency.
- **Logs:** access pattern misses.
- **Alerts:** per-table throttling spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define access patterns per table
  - [ ] Keep GSIs minimal

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Creating many tables for minor variations.
  - **Why it’s bad:** operational sprawl.
  - **Better approach:** consolidate where access patterns overlap.

## Interview readiness
### “Explain it like I’m teaching”
- Multi-table design in DynamoDB keeps each entity in its own table, which is simpler but less flexible. Single-table is more powerful but complex.

### Trap questions (with answers)
1) **Q:** Is multi-table always simpler?
   - **A:** often, but it can still be complex if access patterns overlap.
2) **Q:** Does multi-table eliminate data duplication?
   - **A:** no; you may still duplicate for access patterns.
3) **Q:** Is single-table required for scale?
   - **A:** no; multi-table can scale too.

### Quick self-check (5 items)
- [ ] I can define multi-table design.
- [ ] I can compare it with single-table.
- [ ] I can name a trade-off.
- [ ] I can describe a use case.
- [ ] I can identify a pitfall.

## Links (NO duplication)
### Prerequisites
- [NoSQL access patterns](nosql-access-patterns.md)

### Related topics
- [DynamoDB single-table design](dynamodb-single-table.md)

### Compare with
- [DynamoDB single-table design](dynamodb-single-table.md) — flexibility vs simplicity.
