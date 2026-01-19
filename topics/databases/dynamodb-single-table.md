---
id: dynamodb-single-table
title: "DynamoDB Single-Table Design"
type: concept
status: learning
importance: 60
difficulty: hard
tags: [database, dynamodb, nosql]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# DynamoDB Single-Table Design

## TL;DR (BLUF)
- Single-table design stores multiple entity types in one table to enable diverse access patterns.
- Use it when you need predictable performance and access-pattern-driven modeling.
- Trade-off: higher modeling complexity and careful key design.

## Definition
**What it is:** A DynamoDB modeling approach that uses a single table with multiple entity types and composite keys.
**Key terms:** partition key, sort key, item types, access patterns.

## Why it matters
- It enables efficient queries without joins.
- Poor design leads to hot partitions or unqueryable data.

## Scope & Non-goals
**In scope:** modeling multiple entity types and access patterns in one table.
**Out of scope / NOT solved by this:** ad-hoc queries or relational joins.

## Mental model / Intuition
- Think of a single table as a graph of entities indexed by access paths.

## Decision rules (When to use / When not to use)
### Use it when
- You can define access patterns clearly and need low latency.
- You want to minimize the number of tables.
### Avoid it when
- You need flexible queries across unrelated entities.
- Your access patterns are not stable.

## How I would use it (practical)
- **Context:** User, order, and payment entities in one table.
- **Steps:** define access patterns → design PK/SK → encode entity types → test queries.
- **What success looks like:** all access patterns served by PK/SK or GSIs.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** fast queries, fewer tables, scalable.
- **Cons / Risks:** complex schema, harder debugging.
### Alternatives
- **Multi-table design:** simpler but more tables.
- **How to choose:** choose single-table when access patterns are well-defined and performance is critical.

## Failure modes & Pitfalls
- Hot partitions from unbalanced keys.
- Confusing item types leading to query mistakes.

## Observability (How to detect issues)
- **Metrics:** throttling, partition skew, latency p95.
- **Logs:** access pattern misses.
- **Alerts:** rising throttles or p95 latency.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] List access patterns
  - [ ] Design PK/SK for each
  - [ ] Encode entity type in keys

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Creating single-table without documented access patterns.
  - **Why it’s bad:** the table becomes unqueryable.
  - **Better approach:** start from access patterns and design keys.

## Interview readiness
### “Explain it like I’m teaching”
- Single-table design in DynamoDB puts multiple entity types into one table so all queries are key-based. It’s powerful but demands careful access-pattern planning.

### Trap questions (with answers)
1) **Q:** Is single-table always better than multi-table?
   - **A:** no; it adds complexity and only helps with defined access patterns.
2) **Q:** Can you add new access patterns without changes?
   - **A:** not always; you may need new GSIs or key changes.
3) **Q:** Does single-table mean no duplication?
   - **A:** no; you often duplicate data for access patterns.

### Quick self-check (5 items)
- [ ] I can define single-table design.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can explain key design.
- [ ] I can describe a failure mode.

## Links (NO duplication)
### Prerequisites
- [NoSQL access patterns](nosql-access-patterns.md)

### Related topics
- [DynamoDB](dynamodb.md)

### Compare with
- [DynamoDB multi-table design](dynamodb-multi-table.md)
