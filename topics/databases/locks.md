---
id: locks
title: "Locks"
type: concept
status: learning
importance: 60
difficulty: medium
tags: [database, concurrency]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Locks

## TL;DR (BLUF)
- Locks protect data consistency by controlling concurrent access.
- Use them implicitly via transactions and isolation levels.
- Trade-off: locks can reduce concurrency and cause waits.

## Definition
**What it is:** A mechanism to prevent conflicting operations on the same data.
**Key terms:** row lock, table lock, lock wait, isolation level.

## Why it matters
- Locks keep data consistent under concurrent writes.
- Excessive locking causes contention and latency spikes.

## Scope & Non-goals
**In scope:** lock basics and operational impact.
**Out of scope / NOT solved by this:** distributed locking across services.

## Mental model / Intuition
- Think of locks as “reserved access” to a resource.

## Decision rules (When to use / When not to use)
### Use it when
- You need strict consistency in concurrent updates.
### Avoid it when
- Read-heavy workloads can use snapshot isolation or optimistic approaches.

## How I would use it (practical)
- **Context:** Updating a shared balance row.
- **Steps:** wrap update in transaction → keep it short → monitor waits.
- **What success looks like:** minimal lock waits with consistent updates.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** strong correctness guarantees.
- **Cons / Risks:** contention and deadlocks.
### Alternatives
- **Optimistic concurrency control:** version checks and retries.
- **How to choose:** locks for strict correctness; optimistic control for high concurrency.

## Failure modes & Pitfalls
- Lock waits due to long transactions.
- Deadlocks between competing transactions.

## Observability (How to detect issues)
- **Metrics:** lock wait time, blocked queries.
- **Logs:** lock timeout errors.
- **Alerts:** rising lock waits or timeouts.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Keep transactions short
  - [ ] Avoid unnecessary locking reads

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Locking more rows than needed.
  - **Why it’s bad:** reduces concurrency.
  - **Better approach:** narrow predicates and short transactions.

## Interview readiness
### “Explain it like I’m teaching”
- Locks coordinate concurrent access so updates don’t conflict. They protect correctness but can slow things down when transactions are long or frequent.

### Trap questions (with answers)
1) **Q:** Are locks always bad for performance?
   - **A:** no; they’re necessary for correctness but should be minimized.
2) **Q:** Do locks only apply to writes?
   - **A:** no; some reads also take locks depending on isolation.
3) **Q:** Can you avoid locks entirely?
   - **A:** not if you need strict consistency.

### Quick self-check (5 items)
- [ ] I can define locks precisely.
- [ ] I can describe a trade-off.
- [ ] I can name a failure mode.
- [ ] I can explain how to reduce lock contention.
- [ ] I can relate locks to transactions.

## Links (NO duplication)
### Prerequisites
- [Transactions](transactions.md)

### Related topics
- [Deadlocks](deadlocks.md)

### Compare with
- [Optimistic concurrency control](optimistic-concurrency-control.md)
