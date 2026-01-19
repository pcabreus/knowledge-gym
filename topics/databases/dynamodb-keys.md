---
id: dynamodb-keys
title: "DynamoDB Keys (PK/SK)"
type: concept
status: learning
importance: 60
difficulty: medium
tags: [database, dynamodb, nosql]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# DynamoDB Keys (PK/SK)

## TL;DR (BLUF)
- Partition key (PK) and sort key (SK) define item placement and query access.
- Use composite keys to support multiple access patterns.
- Trade-off: poor key design causes hot partitions and limited queries.

## Definition
**What it is:** The primary key schema in DynamoDB: PK for partitioning and optional SK for sorting.
**Key terms:** partition key, sort key, composite key, access pattern.

## Why it matters
- Key design determines scalability and query capability.
- Bad keys cause throttling and uneven load.

## Scope & Non-goals
**In scope:** PK/SK fundamentals and design principles.
**Out of scope / NOT solved by this:** secondary indexes and streams.

## Mental model / Intuition
- PK chooses the bucket; SK orders items within it.

## Decision rules (When to use / When not to use)
### Use it when
- You need predictable performance and key-based access.
### Avoid it when
- You require ad-hoc queries across many attributes.

## How I would use it (practical)
- **Context:** User activity logs.
- **Steps:** choose PK for user → SK for timestamp → query by time range.
- **What success looks like:** balanced partitions and efficient range queries.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** efficient key-based queries.
- **Cons / Risks:** rigid access patterns.
### Alternatives
- **GSI/LSI:** for secondary access patterns.
- **How to choose:** start with PK/SK for core access patterns, then add indexes.

## Failure modes & Pitfalls
- Hot partitions from monotonic keys.
- Overloaded PK leading to large partitions.

## Observability (How to detect issues)
- **Metrics:** throttles, partition skew, latency.
- **Logs:** hot key access patterns.
- **Alerts:** sustained throttling.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Distribute PK values evenly
  - [ ] Use SK for range queries

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using a single static PK for all items.
  - **Why it’s bad:** creates a hot partition.
  - **Better approach:** partition by a high-cardinality key.

## Interview readiness
### “Explain it like I’m teaching”
- DynamoDB uses partition keys to distribute data and sort keys to order items within a partition. Good key design is everything for performance and scalability.

### Trap questions (with answers)
1) **Q:** Can a table exist without a sort key?
   - **A:** yes; it can have only a partition key.
2) **Q:** Does a good PK guarantee good performance?
   - **A:** not if it creates hot partitions or skew.
3) **Q:** Can you query by non-key attributes without indexes?
   - **A:** not efficiently; you need GSIs/LSIs.

### Quick self-check (5 items)
- [ ] I can define PK and SK.
- [ ] I can explain why keys matter.
- [ ] I can describe a hot partition.
- [ ] I can give a key design example.
- [ ] I can name a trade-off.

## Links (NO duplication)
### Prerequisites
- [NoSQL access patterns](nosql-access-patterns.md)

### Related topics
- [DynamoDB](dynamodb.md)
- [DynamoDB GSI/LSI](dynamodb-gsi-lsi.md)

### Compare with
- [PostgreSQL](postgresql.md) — relational keys vs access-pattern keys.
