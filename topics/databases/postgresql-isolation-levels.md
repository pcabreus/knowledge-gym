---
id: postgresql-isolation-levels
title: "PostgreSQL Isolation Levels"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [database, postgresql, transactions]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# PostgreSQL Isolation Levels

## TL;DR (BLUF)
- Isolation controls how transactions see concurrent changes.
- Use higher isolation for correctness; accept lower for throughput.
- Trade-off: stronger isolation can increase retries and contention.

## Definition
**What it is:** Transaction visibility rules: Read Committed, Repeatable Read, Serializable.
**Key terms:** isolation level, serialization failures, anomalies.

## Why it matters
- It determines correctness guarantees under concurrency.
- Wrong isolation can lead to subtle bugs.

## Scope & Non-goals
**In scope:** Postgres isolation levels and trade-offs.
**Out of scope / NOT solved by this:** distributed transaction isolation.

## Mental model / Intuition
- Think of isolation as how stable your snapshot is during a transaction.

## Decision rules (When to use / When not to use)
### Use it when
- You need stronger correctness (e.g., financial operations).
### Avoid it when
- Throughput is more important than strict isolation.

## How I would use it (practical)
- **Context:** Account balance transfers.
- **Steps:** use Serializable or Repeatable Read → handle retries → keep transactions short.
- **What success looks like:** correctness with manageable retries.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** stronger correctness.
- **Cons / Risks:** higher contention and retries.
### Alternatives
- **Application-level checks:** when isolation is too costly.
- **How to choose:** use the lowest isolation that still guarantees correctness.

## Failure modes & Pitfalls
- Serialization failures not handled with retries.
- Assuming Repeatable Read prevents all anomalies.

## Observability (How to detect issues)
- **Metrics:** serialization failures, retry rate.
- **Logs:** transaction aborts.
- **Alerts:** spikes in serialization failures.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Choose the minimal safe isolation
  - [ ] Implement retry logic for serialization errors

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Running all transactions at Serializable.
  - **Why it’s bad:** unnecessary contention.
  - **Better approach:** only raise isolation when needed.

## Interview readiness
### “Explain it like I’m teaching”
- Isolation levels define how stable your view of the data is while a transaction runs. Postgres defaults to Read Committed; higher levels give stronger guarantees but can increase retries.

### Trap questions (with answers)
1) **Q:** Does Repeatable Read prevent all anomalies?
   - **A:** no; serializable is the strongest and still can fail with retries.
2) **Q:** Are serialization failures bugs?
   - **A:** no; they’re expected and should be retried.
3) **Q:** Is Read Committed always safe?
   - **A:** no; some workflows require stronger guarantees.

### Quick self-check (5 items)
- [ ] I can name the Postgres isolation levels.
- [ ] I can explain a trade-off.
- [ ] I can describe retry needs.
- [ ] I can name a pitfall.
- [ ] I can tie isolation to MVCC.

## Links (NO duplication)
### Prerequisites
- [Transactions](transactions.md)
- [PostgreSQL MVCC](postgresql-mvcc.md)

### Related topics
- [Deadlocks](deadlocks.md)

### Compare with
- [Consistency models](consistency-models.md)
