---
id: dynamodb-hot-partitions
title: "DynamoDB Hot Partitions"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [database, dynamodb, performance]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# DynamoDB Hot Partitions

## TL;DR (BLUF)
- DynamoDB hot partitions are a DynamoDB-specific case of [Hot partitions](hot-partitions.md).
- Use high-cardinality keys and sharding patterns to spread load.
- Trade-off: more complex key design and query fan-out.

## Definition
**What it is:** A DynamoDB-specific manifestation of partition skew where a small set of partition keys dominate traffic.
**Key terms:** partition key, skew, throttling.

## Why it matters
- Hot partitions cause throttling and high latency.
- They often require schema redesign to fix.

## Scope & Non-goals
**In scope:** identifying and avoiding hot partitions.
**Out of scope / NOT solved by this:** tuning unrelated query latency.

## Mental model / Intuition
- Think of all traffic hitting the same checkout lane.

## Decision rules (When to use / When not to use)
### Use it when
- You see throttling or latency for specific keys.
### Avoid it when
- You can design keys to distribute traffic evenly.

## How I would use it (practical)
- **Context:** Partition key is user ID with heavy traffic on a few users.
- **Steps:** add key sharding → spread load → monitor throttling.
- **What success looks like:** throttling drops and latency stabilizes.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** improved throughput and latency.
- **Cons / Risks:** complexity in key design.
### Alternatives
- **Caching:** reduce hot key reads.
- **How to choose:** fix key design first; use caching if needed.

## Failure modes & Pitfalls
- Sharding keys without handling read fan-out.
- Time-based keys causing sudden spikes.

## Observability (How to detect issues)
- **Metrics:** throttling, partition-level latency.
- **Logs:** hot key access patterns.
- **Alerts:** recurring throttles for specific keys.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Analyze key distribution
  - [ ] Add sharding when needed

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using monotonic keys for heavy traffic.
  - **Why it’s bad:** creates hot partitions.
  - **Better approach:** add randomness or sharding.

## Interview readiness
### “Explain it like I’m teaching”
- Hot partitions happen when too many requests hit the same key. DynamoDB then throttles that partition; you fix it by improving key distribution.

### Trap questions (with answers)
1) **Q:** Can provisioned throughput fix hot partitions alone?
   - **A:** no; it doesn’t solve skewed keys.
2) **Q:** Are hot partitions only a write problem?
   - **A:** no; reads can also be hot.
3) **Q:** Can you avoid hot partitions with GSIs alone?
   - **A:** not if the base key is still skewed.

### Quick self-check (5 items)
- [ ] I can define hot partitions.
- [ ] I can explain a mitigation.
- [ ] I can name a pitfall.
- [ ] I can describe detection signals.
- [ ] I can relate it to key design.

## Links (NO duplication)
### Prerequisites
- [DynamoDB keys](dynamodb-keys.md)
- [Hot partitions](hot-partitions.md)

### Related topics
- [DynamoDB](dynamodb.md)

### Compare with
- [Read/write contention](read-write-contention.md) — contention in relational systems.
