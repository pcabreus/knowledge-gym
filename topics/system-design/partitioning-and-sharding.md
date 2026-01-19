---
id: partitioning-and-sharding
title: "Partitioning & Sharding"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [system-design, scaling]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Partitioning & Sharding

## TL;DR (BLUF)
- Partitioning splits data across keys; sharding distributes it across nodes.
- Use it when single-node limits are reached.
- Trade-off: more complex queries and operations.

## Definition
**What it is:** Techniques to distribute data across partitions and servers.
**Key terms:** partition key, shard, consistent hashing.

## Why it matters
- It enables horizontal scalability.
- Bad partitioning leads to hot partitions and uneven load.

## Scope & Non-goals
**In scope:** partitioning and sharding basics.
**Out of scope / NOT solved by this:** replication strategies.

## Mental model / Intuition
- Partitioning is splitting; sharding is splitting across machines.

## Decision rules (When to use / When not to use)
### Use it when
- Data volume or throughput exceeds a single node.
### Avoid it when
- You can scale vertically or optimize queries instead.

## How I would use it (practical)
- **Context:** Multi-tenant dataset growing rapidly.
- **Steps:** choose shard key → distribute data → handle rebalancing.
- **What success looks like:** balanced load and predictable latency.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** scalability.
- **Cons / Risks:** operational complexity, cross-shard queries.
### Alternatives
- **Vertical scaling:** simpler but limited.
- **How to choose:** shard when single-node scaling is insufficient.

## Failure modes & Pitfalls
- Hot shards from skewed keys.

## Observability (How to detect issues)
- **Metrics:** shard imbalance, latency p95.
- **Logs:** shard routing errors.
- **Alerts:** uneven shard load.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Choose a good shard key
  - [ ] Plan for resharding

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Sharding by time with skewed traffic.
  - **Why it’s bad:** hot shards.
  - **Better approach:** add hash-based distribution.

## Interview readiness
### “Explain it like I’m teaching”
- Partitioning splits data by key; sharding distributes partitions across machines. It enables scale but makes queries and operations more complex.

### Trap questions (with answers)
1) **Q:** Does sharding remove the need for indexes?
   - **A:** no; each shard still needs indexes.
2) **Q:** Is sharding always faster?
   - **A:** no; cross-shard queries can be slower.
3) **Q:** Can you change shard keys easily?
   - **A:** not usually; it’s costly.

### Quick self-check (5 items)
- [ ] I can define partitioning vs sharding.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain shard key choice.

## Links (NO duplication)
### Prerequisites
- [Distributed systems basics](distributed-systems-basics.md)

### Related topics
- [Hot partitions](../databases/hot-partitions.md)

### Compare with
- [Consistent hashing](../databases/consistency-models.md) — hash vs replica consistency.
