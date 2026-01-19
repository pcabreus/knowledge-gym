---
id: dynamodb
title: "DynamoDB"
type: technology
status: learning
importance: 70
difficulty: medium
tags: [database, nosql, aws]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# DynamoDB

## TL;DR (BLUF)
- Managed NoSQL key-value/document database optimized for predictable low latency at scale.
- Use it when you can design access patterns upfront and need massive scale with minimal ops.
- Trade-off: modeling is access-pattern driven; complex ad-hoc queries are limited.

## Definition
**What it is:** A fully managed NoSQL database with partitioned storage and throughput-based scaling.
**Key terms:** partition key, sort key, GSI/LSI, access patterns, eventual consistency.

## Why it matters
- It delivers predictable performance and scalability with low operational overhead.
- Poor access-pattern design can lead to hot partitions and expensive refactors.

## Scope & Non-goals
**In scope:** key-value access, high scale, predictable latency.
**Out of scope / NOT solved by this:** complex relational joins or ad-hoc analytics.

## Mental model / Intuition
- Think “one table, many access patterns,” modeled around queries, not entities.

## Decision rules (When to use / When not to use)
### Use it when
- You can express data access as key-based lookups and range queries.
- You need high scale with minimal operational burden.
### Avoid it when
- You need complex joins or flexible SQL querying.
- Your access patterns are unclear or change frequently.

## How I would use it (practical)
- **Context:** High-traffic service with key-based lookups.
- **Steps:** define access patterns → design PK/SK → add GSIs → load test hot partitions.
- **What success looks like:** consistent latency with stable partition distribution.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** managed scaling, low latency, high availability.
- **Cons / Risks:** limited query flexibility; costly redesigns for new access patterns.
### Alternatives
- **PostgreSQL:** relational flexibility with higher ops.
- **MongoDB:** more flexible querying, different consistency model.
- **How to choose:** choose DynamoDB when access patterns are stable and scale is critical.

## Failure modes & Pitfalls
- Hot partitions from skewed keys.
- Unexpected throttling due to traffic spikes.

## Observability (How to detect issues)
- **Metrics:** throttle events, consumed RCUs/WCUs, latency p95.
- **Logs:** access patterns causing hot keys.
- **Alerts:** throttling spikes or rising latency.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define access patterns up front
  - [ ] Choose PK/SK to avoid hot partitions
  - [ ] Add GSIs sparingly
- **Performance notes**
  - Batch operations and limit item size.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Modeling entities first, queries later.
  - **Why it’s bad:** leads to expensive redesigns.
  - **Better approach:** start from access patterns.

## Interview readiness
### “Explain it like I’m teaching”
- DynamoDB is a managed NoSQL database optimized for low-latency key-based access at scale. It works best when you can design your schema around known access patterns; otherwise you’ll pay with complexity and refactors.

### Trap questions (with answers)
1) **Q:** Does DynamoDB support ad-hoc SQL joins?
   - **A:** no; it’s key-value/document oriented without joins.
2) **Q:** Can you fix hot partitions without changing keys?
   - **A:** not reliably; you usually need to redesign keys or add sharding.
3) **Q:** Is it always eventually consistent?
   - **A:** no; you can choose strongly consistent reads (with trade-offs).

### Quick self-check (5 items)
- [ ] I can define DynamoDB precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete access-pattern example.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Index](index.md)

### Related topics
- [PostgreSQL](postgresql.md)

### Compare with
- [PostgreSQL](postgresql.md) — relational SQL vs key-value access.
- [MongoDB documents](mongodb-documents.md) — document queries vs access-pattern design.
