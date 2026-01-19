---
id: transactions
title: "Transactions"
type: concept
status: learning
importance: 70
difficulty: medium
tags: [database, sql]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Transactions

## TL;DR (BLUF)
- Transactions group operations into an all-or-nothing unit.
- Use them to maintain consistency across multiple writes.
- Trade-off: longer transactions increase locks and contention.

## Definition
**What it is:** A sequence of operations executed atomically with isolation guarantees.
**Key terms:** ACID, isolation levels, commit, rollback.

## Why it matters
- It prevents partial updates and data corruption.
- Misusing transactions leads to lock contention and deadlocks.

## Scope & Non-goals
**In scope:** atomicity and isolation basics, when to use transactions.
**Out of scope / NOT solved by this:** distributed transactions across systems.

## Mental model / Intuition
- Think of a transaction as a safe container: either everything succeeds or nothing changes.

## Decision rules (When to use / When not to use)
### Use it when
- You need consistency across multiple related writes.
- You must prevent partial state exposure.
### Avoid it when
- You can tolerate eventual consistency and want higher throughput.
- You need cross-service atomicity (use sagas/outbox).

## How I would use it (practical)
- **Context:** Transfer funds between accounts.
- **Steps:** begin transaction → debit → credit → commit → handle rollback on errors.
- **What success looks like:** no partial transfers and consistent balances.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** strong consistency and correctness.
- **Cons / Risks:** contention, deadlocks, reduced throughput.
### Alternatives
- **Outbox pattern:** for cross-service consistency.
- **How to choose:** use transactions for single-database atomicity.

## Failure modes & Pitfalls
- Long transactions causing lock buildup.
- Deadlocks under high concurrency.

## Observability (How to detect issues)
- **Metrics:** lock waits, deadlock count, transaction duration.
- **Logs:** deadlock logs.
- **Alerts:** sustained lock wait spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Keep transactions short
  - [ ] Handle retries on serialization failures

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Holding transactions open across network calls.
  - **Why it’s bad:** lock contention and timeouts.
  - **Better approach:** do external calls outside the transaction.

## Interview readiness
### “Explain it like I’m teaching”
- Transactions make a group of DB operations atomic and isolated. They keep data consistent, but long transactions can slow the system and cause deadlocks.

### Trap questions (with answers)
1) **Q:** Are transactions always required?
   - **A:** no; some workflows can tolerate eventual consistency.
2) **Q:** Do transactions prevent deadlocks?
   - **A:** no; they can still deadlock under concurrency.
3) **Q:** Is a transaction only for writes?
   - **A:** no; reads can be in transactions for consistent snapshots.

### Quick self-check (5 items)
- [ ] I can define a transaction precisely.
- [ ] I can state when to use it.
- [ ] I can describe a trade-off.
- [ ] I can name a failure mode.
- [ ] I can explain isolation at a high level.

## Links (NO duplication)
### Prerequisites
- [ACID properties](acid-properties.md)

### Related topics
- [Locks](locks.md)
- [Deadlocks](deadlocks.md)

### Compare with
- [Outbox pattern](../architecture/outbox-pattern.md)
