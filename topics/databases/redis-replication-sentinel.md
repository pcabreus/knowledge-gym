---
id: redis-replication-sentinel
title: "Redis replication & Sentinel"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [redis, replication, availability]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Redis replication & Sentinel

## TL;DR (BLUF)
- Redis replication provides read replicas and failover support.
- Sentinel manages monitoring and automated failover.
- Trade-off: higher availability vs replication lag and operational complexity.

## Definition
**What it is:** Redis replication copies data from a primary to replicas, while Sentinel monitors nodes and triggers failover when the primary fails.  
**Key terms:** primary/replica, replication lag, failover, Sentinel quorum.

## Why it matters
- It keeps Redis available during node failures.
- It introduces lag and split-brain risk if misconfigured.

## Scope & Non-goals
**In scope:** read scaling, failover automation, replica consistency.  
**Out of scope / NOT solved by this:** cross-region strong consistency or multi-master writes.

## Mental model / Intuition
- A hot standby with a coordinator that decides when to switch.

## Decision rules (When to use / When not to use)
### Use it when
- You need higher availability and can tolerate replica lag.
- You want automated failover without manual intervention.
### Avoid it when
- You need strong consistency on reads.
- You can’t tolerate replica lag or failover complexity.

## How I would use it (practical)
- **Context:** Redis cache in production with uptime requirements.
- **Steps:** configure replicas → deploy Sentinel quorum → test failover → tune client retries.
- **What success looks like:** automatic failover with bounded lag and recovery time.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** higher availability, read scaling.
- **Cons / Risks:** lag, failover tuning, split-brain risk.
### Alternatives
- **Redis Cluster:** adds sharding and built-in failover.
- **Single node:** simpler if availability requirements are low.
- **How to choose:** use replication+Sentinel for HA without sharding; use Cluster if you also need scale out.

## Failure modes & Pitfalls
- Replica lag causing stale reads.
- Failover thrashing due to unstable networks.
- Misconfigured quorum leading to split-brain.

## Observability (How to detect issues)
- **Metrics:** replication lag, sync status, failover count, client errors.
- **Logs:** Sentinel failover decisions and quorum changes.
- **Alerts:** sustained lag or repeated failovers.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Run Sentinel with odd-number quorum.
  - [ ] Configure client timeouts and retry policies.
  - [ ] Test failover regularly.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Reading critical data from replicas without lag checks.
  - **Why it’s bad:** stale reads break correctness.
  - **Better approach:** read from primary or tolerate staleness explicitly.

## Interview readiness
### “Explain it like I’m teaching”
- Redis replication copies data to replicas for availability and read scaling. Sentinel monitors the cluster and automates failover, but you must manage lag and quorum.

### Trap questions (with answers)
1) **Q:** Does replication eliminate data loss?
   - **A:** no; async replication can lose recent writes on failover.
2) **Q:** Is Sentinel the same as Redis Cluster?
   - **A:** no; Sentinel handles failover, Cluster adds sharding and hash slots.
3) **Q:** Are replica reads always safe?
   - **A:** no; lag can return stale data.

### Quick self-check (5 items)
- [ ] I can explain replication vs Sentinel.
- [ ] I can describe replication lag impacts.
- [ ] I can choose between Sentinel and Cluster.
- [ ] I can name a failover pitfall.
- [ ] I can list key HA metrics.

## Links (NO duplication)
### Prerequisites
- [Redis](redis.md)
- [Availability basics](../operations/availability-basics.md)
- [Distributed systems basics](../system-design/distributed-systems-basics.md)

### Related topics
- [Redis scaling (scale up and out)](redis-scaling.md)

### Compare with
- [Partitioning & sharding](../system-design/partitioning-and-sharding.md) — distribution for scale vs HA.

## Notes / Inbox (optional)
- N/A
