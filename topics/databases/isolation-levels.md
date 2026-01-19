---
id: isolation-levels
title: "Isolation Levels (General)"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [database, transactions]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Isolation Levels (General)

## TL;DR (BLUF)
- Isolation levels define visibility of concurrent transactions.
- Use the lowest level that still preserves correctness.
- Trade-off: stronger isolation reduces throughput.

## Definition
**What it is:** Rules that determine what changes a transaction can see.
**Key terms:** read committed, repeatable read, serializable, anomalies.

## Why it matters
- It affects correctness under concurrency.
- Misconfigured isolation leads to subtle bugs.

## Scope & Non-goals
**In scope:** general isolation concepts across DBs.
**Out of scope / NOT solved by this:** vendor-specific edge cases.

## Mental model / Intuition
- Stronger isolation means a more stable snapshot.

## Decision rules (When to use / When not to use)
### Use it when
- You need strict correctness in concurrent updates.
### Avoid it when
- Performance is more important and staleness is acceptable.

## How I would use it (practical)
- **Context:** Financial transfers.
- **Steps:** choose isolation → add retries → monitor failures.
- **What success looks like:** correct balances with manageable retries.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** stronger correctness.
- **Cons / Risks:** more contention and retries.
### Alternatives
- **Optimistic concurrency control:** for low-conflict workloads.
- **How to choose:** use the lowest isolation that still preserves invariants.

## Failure modes & Pitfalls
- Ignoring anomalies like phantom reads.

## Observability (How to detect issues)
- **Metrics:** serialization failures, retry rate.
- **Logs:** transaction aborts.
- **Alerts:** spikes in retries.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Pick minimal safe isolation
  - [ ] Implement retries when needed

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Setting everything to serializable by default.
  - **Why it’s bad:** unnecessary contention.
  - **Better approach:** use stronger isolation only where required.

## Interview readiness
### “Explain it like I’m teaching”
- Isolation levels define how stable a transaction’s view is while others write. Stronger levels give more correctness but reduce throughput.

### Trap questions (with answers)
1) **Q:** Does serializable mean no retries?
   - **A:** no; serialization failures are expected.
2) **Q:** Is read committed always safe?
   - **A:** no; some workflows need stronger guarantees.
3) **Q:** Are isolation levels the same across all DBs?
   - **A:** no; semantics vary.

### Quick self-check (5 items)
- [ ] I can name isolation levels.
- [ ] I can describe a trade-off.
- [ ] I can name a pitfall.
- [ ] I can explain retry needs.
- [ ] I can relate isolation to locks or MVCC.

## Links (NO duplication)
### Prerequisites
- [Transactions](transactions.md)

### Related topics
- [PostgreSQL isolation levels](postgresql-isolation-levels.md)

### Compare with
- [Consistency models](consistency-models.md) — transaction isolation vs replication consistency.
