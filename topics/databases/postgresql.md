---
id: postgresql
title: "PostgreSQL"
type: technology
status: learning
importance: 80
difficulty: medium
tags: [database, sql]
canonical: true
aliases: ["postgres"]
created_at: 2026-01-19
updated_at: 2026-01-19
---

# PostgreSQL

## TL;DR (BLUF)
- Open-source relational database for SQL workloads with strong constraints and extensibility.
- Use it when you need ACID guarantees, rich queries, and a mature ecosystem.
- Trade-off: tuning and scaling require design; poor indexing can hurt writes.

## Definition
**What it is:** A relational database management system (RDBMS) that supports SQL, ACID transactions, extensions, and advanced query planning.
**Key terms:** ACID, constraints, query planner, extensions, MVCC.

## Why it matters
- It’s a backend standard for correctness, tooling, and reliability.
- Strong constraints + indexes enable consistency and performance for complex queries.

## Scope & Non-goals
**In scope:** relational data modeling, transactions, joins, reporting.
**Out of scope / NOT solved by this:** transparent horizontal scaling and ultra-low global latency without extra architecture.

## Mental model / Intuition
- Think of a strict librarian: data integrity rules are enforced, and the planner chooses the fastest path using indexes.
- Good schema + good indexes → predictable performance.

## Decision rules (When to use / When not to use)
### Use it when
- You need strong consistency, joins, and complex querying.
- You value constraints, migrations, and ecosystem maturity.
### Avoid it when
- Your workload is extremely write-heavy with simple access patterns (a KV store may fit better).
- You need globally low latency without a multi-region architecture.

## How I would use it (practical)
- **Context:** Backend service with relational entities and reporting.
- **Steps:** model schema → define constraints → index based on real queries → add connection pooling → monitor latency/locks.
- **What success looks like:** predictable p95 latency, low lock contention, stable write throughput.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** powerful, flexible, reliable, feature-rich.
- **Cons / Risks:** tuning complexity; bad indexes hurt writes; scaling requires explicit design.
### Alternatives
- **MySQL:** similar, different optimizer/features.
- **DynamoDB:** non-relational, requires access-pattern design.
- **MongoDB:** document model; joins/transactions trade-offs.
- **How to choose:** pick PostgreSQL when relational integrity and SQL flexibility matter most.

## Failure modes & Pitfalls
- Lock contention from long transactions.
- Slow queries from missing/incorrect indexes.
- Table/index bloat from heavy updates without maintenance.

## Observability (How to detect issues)
- **Metrics:** p95 query latency, lock waits, deadlocks, cache hit ratio, bloat.
- **Logs:** slow query logs, deadlock reports.
- **Traces:** spans showing DB time dominating requests.
- **Alerts:** sustained p95 latency, rising lock waits, low cache hit ratio.

## Implementation notes (if applicable)
- **Checklist**
   - [ ] Define constraints (FK, unique)
   - [ ] Design indexes based on real queries
   - [ ] Measure with EXPLAIN
   - [ ] Set up backups and replicas
- **Security / Compliance notes**
   - Follow least-privilege DB roles.
- **Performance notes**
   - Keep transactions short; avoid unnecessary indexes.
- **Operational notes**
   - Monitor bloat and vacuum behavior.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Modeling everything as JSON without need.
   - **Why it’s bad:** you lose constraints and clarity.
   - **Better approach:** model stable fields as columns and use [JSONB](jsonb.md) only for true flexibility.
- **Anti-pattern:** Adding indexes “just in case.”
   - **Why it’s bad:** penalizes writes and complicates maintenance.
   - **Better approach:** index based on real query patterns.

## Interview readiness
### “Explain it like I’m teaching”
- PostgreSQL is a relational database that guarantees ACID transactions and rich SQL querying. It’s great when you need strong data integrity and complex joins, but it requires careful schema and index design, and scaling takes deliberate architecture.

### Trap questions (with answers)
1) **Q:** Why can an index make an INSERT slower?
    - **A:** each write must maintain/update the index, increasing work and contention.
2) **Q:** Is JSONB always better than normal columns?
    - **A:** no; JSONB sacrifices constraints and clarity if used for stable structured data.
3) **Q:** Does Postgres scale horizontally by itself?
    - **A:** not transparently; it needs partitioning, replicas, or external sharding.

### Quick self-check (5 items)
- [ ] I can define PostgreSQL in 2–3 sentences.
- [ ] I can explain when to use it and when not to.
- [ ] I can name at least 2 trade-offs.
- [ ] I can give a concrete usage scenario.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Index](index.md)

### Related topics
- [JSONB](jsonb.md)

### Compare with
- [DynamoDB](dynamodb.md) — key-value at scale vs relational SQL.
- [MySQL](mysql.md) — similar RDBMS with different features/optimizer.
- [MongoDB documents](mongodb-documents.md) — document model vs relational constraints.
