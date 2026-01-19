---
id: event-driven-basics
title: "Event-Driven Basics"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [architecture, messaging]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Event-Driven Basics

## TL;DR (BLUF)
- Event-driven systems communicate by emitting and reacting to events.
- Use them for decoupling and asynchronous workflows.
- Trade-off: eventual consistency and debugging complexity.

## Definition
**What it is:** A system design where services publish events and subscribers react.
**Key terms:** event, producer, consumer, async.

## Why it matters
- It reduces coupling and improves scalability.
- It complicates ordering and consistency.

## Scope & Non-goals
**In scope:** core concepts and trade-offs.
**Out of scope / NOT solved by this:** broker-specific internals.

## Mental model / Intuition
- Think of events as notifications of state changes.

## Decision rules (When to use / When not to use)
### Use it when
- You need async processing and loose coupling.
### Avoid it when
- You need strict synchronous consistency.

## How I would use it (practical)
- **Context:** Order created triggers email and billing.
- **Steps:** emit event → consume → handle retries and idempotency.
- **What success looks like:** reliable workflows without tight coupling.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** decoupling and scalability.
- **Cons / Risks:** eventual consistency and harder debugging.
### Alternatives
- **Synchronous calls:** simpler but tightly coupled.
- **How to choose:** use event-driven when async and decoupling matter.

## Failure modes & Pitfalls
- Duplicate events and out-of-order processing.

## Observability (How to detect issues)
- **Metrics:** lag, error rate, retry rate.
- **Logs:** event processing errors.
- **Alerts:** rising lag or retries.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Design idempotent consumers
  - [ ] Monitor lag

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using events for request/response.
  - **Why it’s bad:** adds latency and complexity.
  - **Better approach:** use synchronous calls where appropriate.

## Interview readiness
### “Explain it like I’m teaching”
- Event-driven systems emit events when something happens, and other services react. It improves decoupling but adds eventual consistency and operational complexity.

### Trap questions (with answers)
1) **Q:** Are events always ordered globally?
   - **A:** no; ordering is usually per partition.
2) **Q:** Are events always exactly-once?
   - **A:** no; at-least-once is common.
3) **Q:** Can you avoid idempotency?
   - **A:** no; duplicates happen.

### Quick self-check (5 items)
- [ ] I can define event-driven architecture.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain observability signals.

## Links (NO duplication)
### Prerequisites
- [Messaging basics](messaging-basics.md)

### Related topics
- [Change Data Capture (CDC)](../databases/change-data-capture.md)

### Compare with
- [Request-response architecture](request-response.md)
