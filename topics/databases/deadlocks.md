---
id: deadlocks
title: "Deadlocks"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [database, concurrency]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Deadlocks

## TL;DR (BLUF)
- A deadlock occurs when transactions block each other in a cycle.
- Use short transactions and consistent lock ordering to avoid them.
- Trade-off: higher safety may reduce concurrency.

## Definition
**What it is:** A cyclic wait where each transaction holds a lock needed by another.
**Key terms:** lock ordering, wait graph, rollback.

## Why it matters
- Deadlocks cause transaction failures and retries.
- High deadlock rates signal contention or poor access patterns.

## Scope & Non-goals
**In scope:** DB deadlocks and mitigation patterns.
**Out of scope / NOT solved by this:** distributed deadlocks across services.

## Mental model / Intuition
- Think of two people holding keys to each other’s locks.

## Decision rules (When to use / When not to use)
### Use it when
- You need strict transactional consistency and can tolerate retries.
### Avoid it when
- You can redesign access to reduce lock overlap.

## How I would use it (practical)
- **Context:** Concurrent updates on shared rows.
- **Steps:** enforce consistent lock ordering → keep transactions short → retry on deadlock.
- **What success looks like:** deadlock rate near zero.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** preserves correctness under concurrency.
- **Cons / Risks:** retries and latency spikes.
### Alternatives
- **Optimistic concurrency control:** version checks.
- **How to choose:** use deadlock retries when strict consistency is required.

## Failure modes & Pitfalls
- Long transactions holding locks for too long.
- Inconsistent lock ordering across code paths.

## Observability (How to detect issues)
- **Metrics:** deadlock count, transaction retries.
- **Logs:** deadlock detector logs.
- **Alerts:** spikes in deadlock errors.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Keep transactions short
  - [ ] Order lock acquisition consistently
  - [ ] Retry with backoff

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Retrying without fixing root cause.
  - **Why it’s bad:** hides contention.
  - **Better approach:** reduce lock overlap.

## Interview readiness
### “Explain it like I’m teaching”
- A deadlock is when two transactions each hold a lock the other needs, so neither can proceed. Databases detect this and abort one transaction; you should keep transactions short and lock rows in a consistent order.

### Trap questions (with answers)
1) **Q:** Can deadlocks happen without writes?
   - **A:** yes; some reads take locks depending on isolation.
2) **Q:** Are deadlocks a bug in the database?
   - **A:** no; they’re a concurrency reality with locking.
3) **Q:** Can you ignore deadlocks?
   - **A:** no; you must handle retries and reduce contention.

### Quick self-check (5 items)
- [ ] I can define deadlocks precisely.
- [ ] I can name a cause.
- [ ] I can explain a mitigation.
- [ ] I can identify a signal.
- [ ] I can explain retry strategy.

## Links (NO duplication)
### Prerequisites
- [Locks](locks.md)

### Related topics
- [Transactions](transactions.md)

### Compare with
- [Optimistic concurrency control](optimistic-concurrency-control.md)
