---
id: acid-properties
title: "ACID Properties"
type: concept
status: learning
importance: 65
difficulty: medium
tags: [database, transactions]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# ACID Properties

## TL;DR (BLUF)
- ACID defines transactional guarantees: Atomicity, Consistency, Isolation, Durability.
- Use it to reason about correctness in relational databases.
- Trade-off: stronger guarantees often reduce throughput.

## Definition
**What it is:** A set of guarantees that ensure transactional correctness.
**Key terms:** atomicity, consistency, isolation, durability.

## Why it matters
- It prevents partial writes and data corruption.
- It guides trade-offs between correctness and performance.

## Scope & Non-goals
**In scope:** ACID meaning and implications.
**Out of scope / NOT solved by this:** distributed transactions across systems.

## Mental model / Intuition
- ACID is a safety contract for transactions.

## Decision rules (When to use / When not to use)
### Use it when
- You need strong correctness guarantees.
### Avoid it when
- Eventual consistency is acceptable and throughput is more important.

## How I would use it (practical)
- **Context:** Banking or financial workflows.
- **Steps:** use transactions → pick isolation level → verify durability.
- **What success looks like:** no partial updates or lost data.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** strong correctness.
- **Cons / Risks:** reduced throughput and higher contention.
### Alternatives
- **Eventual consistency:** for high-scale systems.
- **How to choose:** prioritize ACID for correctness-critical systems.

## Failure modes & Pitfalls
- Misunderstanding isolation leading to anomalies.

## Observability (How to detect issues)
- **Metrics:** transaction failures, lock waits.
- **Logs:** transaction aborts.
- **Alerts:** spikes in aborts or deadlocks.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Choose appropriate isolation level
  - [ ] Keep transactions short

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Assuming ACID across microservices.
  - **Why it’s bad:** cross-system consistency isn’t automatic.
  - **Better approach:** use sagas or outbox patterns.

## Interview readiness
### “Explain it like I’m teaching”
- ACID describes how transactions behave: they’re atomic, consistent, isolated, and durable. It’s the baseline for correctness in relational databases.

### Trap questions (with answers)
1) **Q:** Is ACID guaranteed across services?
   - **A:** no; it’s typically within a single database.
2) **Q:** Does ACID mean high availability?
   - **A:** no; it’s about correctness, not availability.
3) **Q:** Can you relax isolation for performance?
   - **A:** yes; isolation levels trade correctness for throughput.

### Quick self-check (5 items)
- [ ] I can define ACID precisely.
- [ ] I can explain each letter.
- [ ] I can state a trade-off.
- [ ] I can name a pitfall.
- [ ] I can explain isolation levels at a high level.

## Links (NO duplication)
### Prerequisites
- [Transactions](transactions.md)

### Related topics
- [Locks](locks.md)

### Compare with
- [Consistency models](consistency-models.md)
