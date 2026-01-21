---
id: saga-pattern
title: "Saga Pattern"
type: pattern
status: learning
importance: 55
difficulty: medium
tags: [architecture, distributed-systems, reliability]
canonical: true
aliases: [saga]
created_at: 2026-01-21
updated_at: 2026-01-21
---

# Saga Pattern

## TL;DR (BLUF)
- A saga coordinates a distributed transaction as a sequence of local transactions with compensations.
- Use it for cross-service workflows where ACID across services is not possible.
- Trade-off: eventual consistency and complex failure handling.

## Definition
**What it is:** A pattern for managing multi-step, cross-service workflows using local transactions and compensating actions to undo partial work.  
**Key terms:** saga, compensation, orchestration, choreography, local transaction, eventual consistency.

## Why it matters
- It provides a reliable way to handle failures in distributed workflows.
- It avoids tight coupling and global locks across services.

## Scope & Non-goals
**In scope:** saga flow, compensation strategy, and reliability trade-offs.  
**Out of scope / NOT solved by this:** strong consistency across services or 2PC equivalents.

## Mental model / Intuition
- Think of a travel booking: flight + hotel + car. If one fails, you cancel the others.
- Each step commits locally; compensations roll back business effects.

## Decision rules (When to use / When not to use)
### Use it when
- You need cross-service workflows without distributed transactions.
- Each step can define a compensating action.
- Eventual consistency is acceptable.
### Avoid it when
- You need strict atomicity across services.
- Compensation is impossible or unsafe.
- The workflow is simple enough for a single service transaction.

## How I would use it (practical)
- **Context:** Order creation spanning payments, inventory, and shipping services.
- **Steps:**
  1) Model each step as a local transaction.
  2) Define compensations for each step.
  3) Choose orchestration (central coordinator) or choreography (events).
  4) Add timeouts, retries, and a dead-letter queue.
- **What success looks like:** no stranded resources and clear recovery paths.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** avoids distributed locks, scales across services, resilient to partial failure.
- **Cons / Risks:** eventual consistency, complex compensation logic, and harder observability.
### Alternatives
- **[Request-response architecture](request-response.md):** simpler for synchronous, single-service workflows.
- **[Outbox pattern](outbox-pattern.md):** safer event publication for local transactions.
- **How to choose:** use sagas when multi-service coordination and failure recovery matter most.

## Failure modes & Pitfalls
- Compensations not idempotent, causing over-correction.
- Orchestration becoming a single point of failure.
- Choreography leading to hidden coupling and unclear flow ownership.

## Observability (How to detect issues)
- **Metrics:** saga completion time, rollback rate, compensation failures.
- **Logs:** saga ID, step transitions, compensation outcomes.
- **Traces:** end-to-end saga spans with step timing.
- **Alerts:** high rollback rate, stuck sagas, or repeated compensation failures.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define compensations for every step
  - [ ] Ensure idempotency for steps and compensations
  - [ ] Correlate all steps with a saga ID
- **Security / Compliance notes**
  - Ensure compensations preserve audit trails.
- **Performance notes**
  - Avoid long-running locks; use timeouts and async retries.
- **Operational notes**
  - Monitor stuck or long-running sagas.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using sagas to mimic strict ACID across services.
  - **Why it’s bad:** compensations are not real rollbacks and can leave side effects.
  - **Better approach:** redesign workflow or consolidate into one service when possible.

## Interview readiness
### “Explain it like I’m teaching”
- A saga breaks a distributed transaction into local transactions and uses compensating actions to undo partial work. It trades strict consistency for resilience and scalability.

### Trap questions (with answers)
1) **Q:** Are compensations the same as database rollbacks?
   - **A:** No; they are business-level actions that may not fully undo side effects.
2) **Q:** Does a saga guarantee exactly-once execution?
   - **A:** No; steps must be idempotent and resilient to retries.
3) **Q:** Is choreography always better than orchestration?
   - **A:** No; choreography can become opaque and tightly coupled without clear ownership.

### Quick self-check (5 items)
- [ ] I can define the saga pattern precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Event-driven architecture](event-driven-basics.md)
- [Transactions](../databases/transactions.md)
- [ACID properties](../databases/acid-properties.md)

### Related topics
- [Outbox pattern](outbox-pattern.md)
- [Dual-write pattern](dual-write-pattern.md)
- [Change Data Capture (CDC)](../databases/change-data-capture.md)

### Compare with
- [Request-response architecture](request-response.md) — synchronous flow vs distributed workflow.

## Notes / Inbox (optional)
- N/A
