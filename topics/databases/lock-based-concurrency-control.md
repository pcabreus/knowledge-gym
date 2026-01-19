---
id: lock-based-concurrency-control
title: "Lock-Based Concurrency Control"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [database, concurrency]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Lock-Based Concurrency Control

## TL;DR (BLUF)
- Lock-based control uses locks to prevent conflicting operations.
- Use it when conflicts are common and correctness is critical.
- Trade-off: reduced concurrency and potential deadlocks.

## Definition
**What it is:** A concurrency strategy that uses locks to serialize conflicting operations.
**Key terms:** locks, blocking, deadlocks.

## Why it matters
- It provides strong correctness under contention.
- It can reduce throughput and increase latency.

## Scope & Non-goals
**In scope:** lock-based concurrency concepts and trade-offs.
**Out of scope / NOT solved by this:** distributed locking across services.

## Mental model / Intuition
- Only one writer at a time per locked resource.

## Decision rules (When to use / When not to use)
### Use it when
- Conflicts are frequent and correctness is paramount.
### Avoid it when
- Conflicts are rare and optimistic control is more efficient.

## How I would use it (practical)
- **Context:** Inventory decrement under high contention.
- **Steps:** lock row → update → commit quickly.
- **What success looks like:** correct counts with manageable lock waits.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** strong correctness.
- **Cons / Risks:** blocking and deadlocks.
### Alternatives
- **Optimistic concurrency control:** for low-conflict workloads.
- **How to choose:** lock-based for high-conflict scenarios.

## Failure modes & Pitfalls
- Long transactions increasing lock wait times.
- Deadlocks under competing lock order.

## Observability (How to detect issues)
- **Metrics:** lock waits, deadlocks.
- **Logs:** lock timeout errors.
- **Alerts:** rising lock wait time.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Keep transactions short
  - [ ] Use consistent lock ordering

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Locking more data than needed.
  - **Why it’s bad:** reduces concurrency.
  - **Better approach:** narrow the locked scope.

## Interview readiness
### “Explain it like I’m teaching”
- Lock-based concurrency prevents conflicts by blocking access. It’s safe but can reduce throughput and cause deadlocks if not managed carefully.

### Trap questions (with answers)
1) **Q:** Are locks always worse than optimistic control?
   - **A:** no; for high-conflict workloads locks are safer.
2) **Q:** Do locks eliminate the need for retries?
   - **A:** no; deadlocks still require retries.
3) **Q:** Are locks only for writes?
   - **A:** no; some reads also take locks.

### Quick self-check (5 items)
- [ ] I can define lock-based concurrency control.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain lock ordering.

## Links (NO duplication)
### Prerequisites
- [Locks](locks.md)

### Related topics
- [Deadlocks](deadlocks.md)

### Compare with
- [Optimistic concurrency control](optimistic-concurrency-control.md)
