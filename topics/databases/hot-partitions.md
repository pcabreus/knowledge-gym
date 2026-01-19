---
id: hot-partitions
title: "Hot Partitions"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [database, performance, distributed]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Hot Partitions

## TL;DR (BLUF)
- Hot partitions happen when traffic is skewed to a small subset of partitions.
- Use key design, sharding, and caching to spread load.
- Trade-off: more complex keys and query logic.

## Definition
**What it is:** Uneven distribution of read/write load across partitions caused by skewed keys or access patterns.
**Key terms:** partition key, skew, throttling, shard.

## Why it matters
- Hot partitions cause throttling, tail latency spikes, and unstable throughput.
- Fixing them often requires schema or key redesign.

## Scope & Non-goals
**In scope:** detection and mitigation of partition skew.
**Out of scope / NOT solved by this:** global capacity planning.

## Mental model / Intuition
- Think of one checkout line with all customers while other lines are empty.

## Decision rules (When to use / When not to use)
### Use it when
- You see throttling or elevated latency for specific keys.
### Avoid it when
- You can redesign keys to distribute traffic evenly.

## How I would use it (practical)
- **Context:** A few tenants generate most traffic.
- **Steps:** analyze key distribution → add sharding → cache hot reads.
- **What success looks like:** reduced throttling and flatter latency percentiles.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** higher throughput, lower tail latency.
- **Cons / Risks:** increased schema complexity and query fan-out.
### Alternatives
- **Read caching:** reduce hot reads.
- **How to choose:** fix key design first, then cache if needed.

## Failure modes & Pitfalls
- Sharding keys without handling read fan-out.
- Time-based keys creating spikes.

## Observability (How to detect issues)
- **Metrics:** per-partition throughput, throttles, p95 latency.
- **Logs:** hot key access patterns.
- **Alerts:** repeated throttles for specific keys.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Analyze key distribution
  - [ ] Add sharding when needed
  - [ ] Update queries for sharded keys

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using monotonic keys for hot data.
  - **Why it’s bad:** creates traffic skew.
  - **Better approach:** add randomness or shard by hash.

## Interview readiness
### “Explain it like I’m teaching”
- Hot partitions occur when too many requests target the same partition. The fix is to design keys that spread load and sometimes add sharding or caching.

### Trap questions (with answers)
1) **Q:** Can provisioning alone fix hot partitions?
   - **A:** no; skewed keys still overload a single partition.
2) **Q:** Are hot partitions only a write problem?
   - **A:** no; reads can be hot too.
3) **Q:** Does adding more partitions always help?
   - **A:** only if keys distribute load; otherwise the same keys stay hot.

### Quick self-check (5 items)
- [ ] I can define hot partitions.
- [ ] I can explain a mitigation.
- [ ] I can name a pitfall.
- [ ] I can describe detection signals.
- [ ] I can relate it to key design.

## Links (NO duplication)
### Prerequisites
- [Partitioning & sharding](../system-design/partitioning-and-sharding.md)

### Related topics
- [Read/write contention](read-write-contention.md)

### Compare with
- [DynamoDB hot partitions](dynamodb-hot-partitions.md) — DynamoDB-specific guidance.
