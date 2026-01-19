---
id: cassandra
title: "Cassandra"
type: technology
status: learning
importance: 55
difficulty: hard
tags: [database, nosql, distributed]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Cassandra

## TL;DR (BLUF)
- Distributed wide-column database designed for high availability and write scalability.
- Use it for time-series and write-heavy workloads with predictable access patterns.
- Trade-off: data modeling is query-driven and consistency is tunable.

## Definition
**What it is:** A distributed NoSQL database using a wide-column model and tunable consistency.
**Key terms:** partition key, replication factor, tunable consistency, wide-column.

## Why it matters
- It handles large-scale, write-heavy workloads with high availability.
- Poor data modeling leads to hot partitions and inefficiency.

## Scope & Non-goals
**In scope:** write-heavy, globally distributed workloads with simple queries.
**Out of scope / NOT solved by this:** complex joins or ad-hoc analytics.

## Mental model / Intuition
- Think of Cassandra as a distributed, partitioned log optimized for predictable queries.

## Decision rules (When to use / When not to use)
### Use it when
- You need high write throughput and multi-region availability.
- Queries are predictable and key-based.
### Avoid it when
- You need flexible SQL queries or strong consistency by default.
- You can’t design around access patterns.

## How I would use it (practical)
- **Context:** Time-series events with high write rates.
- **Steps:** design partition keys → model tables per query → test hot partitions.
- **What success looks like:** stable write throughput with balanced partitions.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** high availability, linear write scaling.
- **Cons / Risks:** complex modeling; eventual consistency by default.
### Alternatives
- **DynamoDB:** managed key-value store with similar access-pattern design.
- **PostgreSQL:** relational queries with stronger consistency.
- **How to choose:** choose Cassandra for large-scale write-heavy workloads with stable access patterns.

## Failure modes & Pitfalls
- Hot partitions from skewed keys.
- Read amplification from wide rows.

## Observability (How to detect issues)
- **Metrics:** write latency, partition size distribution, compaction metrics.
- **Logs:** timeouts and read/write errors.
- **Alerts:** rising timeouts or compaction backlog.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Design partition keys to spread load
  - [ ] Model tables per query
  - [ ] Monitor compaction and latency

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Designing tables like a relational model.
  - **Why it’s bad:** leads to inefficient queries and hot partitions.
  - **Better approach:** model by access patterns.

## Interview readiness
### “Explain it like I’m teaching”
- Cassandra is a distributed wide-column database built for high write throughput and availability. It’s great for predictable access patterns but requires careful partition-key design and accepts eventual consistency.

### Trap questions (with answers)
1) **Q:** Does Cassandra support joins?
   - **A:** no; it’s query-driven and avoids joins.
2) **Q:** Is Cassandra strongly consistent by default?
   - **A:** no; consistency is tunable and often eventual.
3) **Q:** Can you change the primary key without redesign?
   - **A:** no; it’s central to the data model.

### Quick self-check (5 items)
- [ ] I can define Cassandra precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a time-series example.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Partitioning & sharding](../system-design/partitioning-and-sharding.md)

### Related topics
- [DynamoDB](dynamodb.md)
### Compare with
- [DynamoDB](dynamodb.md) — managed key-value vs self-managed wide-column.
- [PostgreSQL](postgresql.md) — SQL + strong consistency vs tunable consistency.
