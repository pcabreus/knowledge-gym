---
id: postgresql-scaling-playbook
title: "PostgreSQL Scaling Playbook"
type: concept
status: learning
importance: 70
difficulty: medium
tags: [postgresql, scaling, performance, operations]
canonical: true
aliases: []
created_at: 2026-02-05
updated_at: 2026-02-05
---

# PostgreSQL Scaling Playbook

## TL;DR (BLUF)
- Scale in order: optimize → scale up → read replicas → partitioning → sharding.
- Use clear signals (CPU/RAM/IO, latency, bloat, replication lag) to pick the next step.
- Trade-off: each step buys capacity but increases operational complexity.

## Definition
**What it is:** A decision ladder for when and how to scale PostgreSQL, prioritizing tuning before adding nodes.  
**Key terms:** vertical scaling (scale-up), read replica, replication lag, partitioning, sharding, WAL, p95/p99.

## Why it matters
- Avoids premature sharding and the operational cost that follows.
- Keeps latency predictable while controlling cost and risk.

## Scope & Non-goals
**In scope:** single-cluster PostgreSQL scaling decisions and their signals.  
**Out of scope / NOT solved by this:** multi-region active-active replication (see TODO below).

## Mental model / Intuition
- A ladder: each rung costs more to operate, so climb only when the previous rung is exhausted.

## Decision rules (When to use / When not to use)
### Use it when
- You see sustained latency or saturation and need a structured next step.
- You need a defensible scaling plan rather than ad-hoc fixes.

### Avoid it when
- Your workload is small or bursty and can be solved by basic tuning.
- You cannot measure CPU, IO, and latency (fix observability first).

## How I would use it (practical)
- **Context:** A product feed service on PostgreSQL with growing traffic.
- **Steps:**
  1) **Tune first (no scaling):** fix [indexes](index.md), bad [query plans](query-planning.md), avoid N+1 queries, add [connection pooling](connection-pooling.md), tune [autovacuum](postgresql-vacuum-autovacuum.md), reduce [bloat](postgresql-bloat.md).
  2) **Scale up (vertical):** if CPU or RAM is saturated and query tuning is done.
  3) **Read replicas:** if reads dominate and you can tolerate [eventual consistency](consistency-models.md) on reads.
  4) **Partitioning:** if tables are huge, [VACUUM](postgresql-vacuum-autovacuum.md) is struggling, and time-range queries dominate.
  5) **Sharding:** if writes or lock contention exceed a single node and scale-up/replicas no longer help.
- **What success looks like:** p95 latency stabilizes, CPU/IO headroom returns, and maintenance tasks complete on schedule.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** predictable decision path, avoids premature distributed complexity.
- **Cons / Risks:** replicas add lag; partitioning/sharding increase query and ops complexity.

### Alternatives
- **Queue-based async writes:** reduce synchronous DB pressure.
- **Caching:** offload reads for hot paths.
- **How to choose:** if the bottleneck is read-heavy, prefer replicas/caching; if write-heavy, plan for sharding or workload redesign.

## Failure modes & Pitfalls
- Scaling before fixing [slow queries](slow-query-diagnosis.md).
- Misreading 4xx or app timeouts as DB saturation.
- Relying on replicas while ignoring replication lag for read-your-writes paths.
- Partitioning by a skewed key → [hot partitions](hot-partitions.md).

## Observability (How to detect issues)
- **Metrics:** CPU, iowait, cache hit ratio, p95/p99 latency, WAL rate, replication lag.
- **Logs:** autovacuum and checkpoint warnings, slow query logs.
- **Traces:** hot endpoints with high DB span time.
- **Alerts:** sustained p95 degradation + saturation for >N minutes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Confirm query plans and indexes first
  - [ ] Measure CPU/RAM/IO saturation and cache hit ratio
  - [ ] Add read replicas only for read-heavy paths
  - [ ] Partition by access pattern (time/tenant), not by convenience
  - [ ] Shard only after scale-up and replicas are exhausted

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Shard first because “Postgres doesn’t scale.”
  - **Why it’s bad:** you inherit distributed complexity without fixing root causes.
  - **Better approach:** tune → scale up → replicas → partitioning → shard.

## Interview readiness
### “Explain it like I’m teaching”
- PostgreSQL scaling is a ladder: fix queries and maintenance first, then scale up, then add read replicas, then partition large tables, and only then shard if a single node can’t keep up with writes.

### Trap questions (with answers)
1) **Q:** Should you add replicas before fixing slow queries?
   - **A:** No; replicas copy inefficiency. Fix query plans and indexes first.
2) **Q:** Do read replicas help write bottlenecks?
   - **A:** Not really; they help read-heavy workloads, not write throughput.
3) **Q:** Is partitioning the same as sharding?
   - **A:** No; partitioning splits data inside one database, sharding distributes it across nodes.

### Quick self-check (5 items)
- [ ] I can list the scaling ladder in order.
- [ ] I can name the signals for each step.
- [ ] I can explain replication lag risks.
- [ ] I can explain when partitioning helps.
- [ ] I can justify when sharding is unavoidable.

## Links (NO duplication)
### Prerequisites
- [PostgreSQL](postgresql.md)
- [Index](index.md)
- [Query planning](query-planning.md)
- [Slow query diagnosis](slow-query-diagnosis.md)
- [Connection pooling](connection-pooling.md)
- [PostgreSQL vacuum & autovacuum](postgresql-vacuum-autovacuum.md)
- [PostgreSQL bloat](postgresql-bloat.md)
- [PostgreSQL statistics & ANALYZE](postgresql-stats-and-analyze.md)
- (TODO) PostgreSQL replication basics
- (TODO) N+1 query problem

### Related topics
- [Partitioning & sharding](../system-design/partitioning-and-sharding.md)
- [Hot partitions](hot-partitions.md)
- [Read/write contention](read-write-contention.md)
- [Capacity planning](../operations/capacity-planning.md)

### Compare with
- [Redis scaling (scale up and out)](redis-scaling.md) — different trade-offs for in-memory systems.

## Notes / Inbox (optional)
- N/A
