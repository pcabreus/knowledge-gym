---
id: nosql-access-patterns
title: "NoSQL Access Patterns"
type: concept
status: learning
importance: 60
difficulty: medium
tags: [database, nosql]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# NoSQL Access Patterns

## TL;DR (BLUF)
- NoSQL data modeling starts from access patterns, not entities.
- Use it when you can define queries upfront and need scale.
- Trade-off: new queries often require new tables or indexes.

## Definition
**What it is:** Designing data structures around known query patterns in NoSQL systems.
**Key terms:** access pattern, denormalization, partition key.

## Why it matters
- It prevents expensive redesigns and hot partitions.
- Misaligned access patterns cause performance bottlenecks.

## Scope & Non-goals
**In scope:** access-pattern-driven modeling across NoSQL systems.
**Out of scope / NOT solved by this:** key-level mechanics and query syntax details.

## Mental model / Intuition
- Start with “How will I query?” then build the table for that query.

## Decision rules (When to use / When not to use)
### Use it when
- You know the critical queries upfront.
- You need predictable scale and latency.
### Avoid it when
- Queries are unknown or constantly changing.

## How I would use it (practical)
- **Context:** DynamoDB table for user activity.
- **Steps:** list access patterns → design PK/SK → add GSIs for secondary queries.
- **What success looks like:** stable latency and no hot partitions.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** scalable and predictable performance.
- **Cons / Risks:** rigid queries and redesign cost.
### Alternatives
- **PostgreSQL:** flexible SQL querying.
- **How to choose:** use NoSQL access patterns when scale and predictability matter more than flexibility.

## Failure modes & Pitfalls
- Hot partitions from skewed keys.
- Overloading a single table with unrelated access patterns.

## Observability (How to detect issues)
- **Metrics:** partition skew, throttling, latency.
- **Logs:** access patterns causing hot keys.
- **Alerts:** rising throttles or p95 latency.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] List access patterns explicitly
  - [ ] Design PK/SK for each
  - [ ] Validate with load tests

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Modeling entities first, queries later.
  - **Why it’s bad:** leads to expensive refactors.
  - **Better approach:** start from access patterns.

## Interview readiness
### “Explain it like I’m teaching”
- NoSQL access patterns mean you design the data model around how you query. It’s great for scale but requires knowing your queries up front.

### Trap questions (with answers)
1) **Q:** Can you add new queries for free?
   - **A:** no; you usually add new indexes or tables.
2) **Q:** Is NoSQL always faster than SQL?
   - **A:** no; it depends on access patterns and workload.
3) **Q:** Do access patterns matter in DynamoDB?
   - **A:** yes; they are central to the design.

### Quick self-check (5 items)
- [ ] I can define access-pattern modeling.
- [ ] I can explain when to use it.
- [ ] I can name a failure mode.
- [ ] I can describe a design step.
- [ ] I can compare with SQL.

## Links (NO duplication)
### Prerequisites
- [Key-value data modeling](key-value-data-modeling.md)

### Related topics
- [DynamoDB](dynamodb.md)
- [Cassandra](cassandra.md)
- [Key-value data modeling](key-value-data-modeling.md)

### Compare with
- [PostgreSQL](postgresql.md) — flexible SQL vs access-pattern design.
