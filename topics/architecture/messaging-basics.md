---
id: messaging-basics
title: "Messaging Basics"
type: concept
status: learning
importance: 45
difficulty: medium
tags: [architecture, messaging]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Messaging Basics

## TL;DR (BLUF)
- Messaging enables async communication between services.
- Use it to decouple producers and consumers.
- Trade-off: eventual consistency and operational complexity.

## Definition
**What it is:** Asynchronous communication via queues or topics.
**Key terms:** queue, topic, producer, consumer.

## Why it matters
- It improves resilience and scalability.
- It adds latency and complexity.

## Scope & Non-goals
**In scope:** messaging concepts and trade-offs.
**Out of scope / NOT solved by this:** broker-specific tuning.

## Mental model / Intuition
- A message is a unit of work handed to another service.

## Decision rules (When to use / When not to use)
### Use it when
- You need async processing or decoupling.
### Avoid it when
- You need synchronous guarantees.

## How I would use it (practical)
- **Context:** Email sending after user signup.
- **Steps:** enqueue message → consumer sends email → handle retries.
- **What success looks like:** reliable processing with low lag.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** decoupling and resilience.
- **Cons / Risks:** ordering and duplicate handling.
### Alternatives
- **Request-response:** synchronous calls.
- **How to choose:** use messaging for async workflows.

## Failure modes & Pitfalls
- Duplicate messages without idempotency.

## Observability (How to detect issues)
- **Metrics:** queue depth, consumer lag.
- **Logs:** processing errors.
- **Alerts:** growing backlog.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Make consumers idempotent
  - [ ] Monitor lag

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using messaging for request/response.
  - **Why it’s bad:** adds latency and complexity.
  - **Better approach:** use direct calls for synchronous needs.

## Interview readiness
### “Explain it like I’m teaching”
- Messaging lets services communicate asynchronously. It’s great for decoupling, but you must handle retries, ordering, and duplicates.

### Trap questions (with answers)
1) **Q:** Are messages always delivered exactly once?
   - **A:** no; at-least-once is common.
2) **Q:** Are queues and topics the same?
   - **A:** no; queues deliver to one consumer, topics to many.
3) **Q:** Can you skip idempotency?
   - **A:** no; duplicates happen.

### Quick self-check (5 items)
- [ ] I can define messaging basics.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain queue vs topic.

## Links (NO duplication)
### Prerequisites
- N/A

### Related topics
- [Event-driven basics](event-driven-basics.md)

### Compare with
- [Request-response architecture](request-response.md) — async vs sync.
