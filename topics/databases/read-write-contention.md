---
id: read-write-contention
title: "Read/Write Contention"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [database, performance, concurrency]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Read/Write Contention

## TL;DR (BLUF)
- Contention happens when concurrent reads and writes compete for the same resources.
- Use indexing and shorter transactions to reduce contention.
- Trade-off: more indexes can slow writes.

## Definition
**What it is:** Performance degradation caused by concurrent access to the same rows, indexes, or locks.
**Key terms:** lock contention, hot rows, write amplification.

## Why it matters
- Contention can dominate latency even with good hardware.
- It often appears only at scale.

## Scope & Non-goals
**In scope:** contention sources and mitigation patterns.
**Out of scope / NOT solved by this:** network-level bottlenecks.

## Mental model / Intuition
- Think of a busy checkout lane; everyone queues behind a shared lock.

## Decision rules (When to use / When not to use)
### Use it when
- You observe lock waits or high write latency.
### Avoid it when
- The workload is naturally partitionable and can be sharded.

## How I would use it (practical)
- **Context:** Hot counter row updated by many requests.
- **Steps:** reduce hot row updates → shard counters → shorten transactions.
- **What success looks like:** lower lock waits and stable latency.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** improved throughput.
- **Cons / Risks:** added complexity (sharding, retries).
### Alternatives
- **Caching:** offload reads.
- **How to choose:** mitigate contention before scaling hardware.

## Failure modes & Pitfalls
- Hot rows from monotonically increasing IDs.
- Lock escalation from large updates.

## Observability (How to detect issues)
- **Metrics:** lock waits, write latency, deadlocks.
- **Logs:** slow queries with lock waits.
- **Alerts:** rising lock wait time.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Identify hot rows/indexes
  - [ ] Reduce transaction scope
  - [ ] Consider sharding or batching

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Single global counter for high-traffic updates.
  - **Why it’s bad:** creates a hot row.
  - **Better approach:** shard counters or use approximations.

## Interview readiness
### “Explain it like I’m teaching”
- Read/write contention is when many operations fight over the same rows or locks, slowing everything down. You fix it by reducing lock time, sharding, and indexing.

### Trap questions (with answers)
1) **Q:** Do more indexes always reduce contention?
   - **A:** no; indexes can increase write cost and contention.
2) **Q:** Can contention be solved only with bigger hardware?
   - **A:** no; it’s often a data-access pattern problem.
3) **Q:** Are reads always free?
   - **A:** no; they can block or be blocked depending on isolation.

### Quick self-check (5 items)
- [ ] I can define contention precisely.
- [ ] I can name a mitigation.
- [ ] I can describe a pitfall.
- [ ] I can explain a signal.
- [ ] I can compare with hot partitions.

## Links (NO duplication)
### Prerequisites
- [Locks](locks.md)

### Related topics
- [Deadlocks](deadlocks.md)

### Compare with
- [DynamoDB hot partitions](dynamodb-hot-partitions.md) — key skew in NoSQL.
