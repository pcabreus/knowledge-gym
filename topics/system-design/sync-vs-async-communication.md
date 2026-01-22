---
id: sync-vs-async-communication
title: "Synchronous vs Asynchronous Communication"
type: concept
status: learning
importance: 80
difficulty: medium
tags: [system-design, architecture, reliability]
canonical: true
aliases: []
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Synchronous vs Asynchronous Communication

## TL;DR (BLUF)
- **Synchronous** calls block the caller and require immediate response.
- **Asynchronous** flows decouple services but add eventual consistency and retries.
- Choose based on latency budget, consistency needs, and failure handling.

## Definition
**What it is:** Two communication styles: sync request/response (caller waits) and async messaging (caller does not wait).
**Key terms:** synchronous (synchronous), asynchronous (asynchronous), coupling, latency budget (latency budget), eventual consistency (eventual consistency).

## Why it matters
- Communication style shapes availability, latency, and operational complexity.
- Wrong choice causes slow user flows or brittle distributed systems.

## Scope & Non-goals
**In scope:** architectural decision rules and trade-offs.
**Out of scope / NOT solved by this:** broker internals or API protocol details.

## Mental model / Intuition
- Sync is a phone call; async is sending a message and continuing.

## Decision rules (When to use / When not to use)
### Use synchronous when
- The user needs an immediate result.
- You require strong consistency before proceeding.

### Use asynchronous when
- Work can be deferred or long-running.
- You need fan-out, retries, or decoupling.

## How I would use it (practical)
- **Context:** Case creation must return immediately; notifications can be async.
- **Steps:**
  1) Keep core write path synchronous.
  2) Publish events for downstream side effects.
  3) Make consumers idempotent and add DLQ for poison messages.
- **What success looks like:** fast user latency and resilient side-effects.

## Trade-offs & Alternatives
### Trade-offs
- **Pros (sync):** simple semantics, immediate consistency.
- **Cons (sync):** tight coupling, cascading failures.
- **Pros (async):** decoupling, resilience, scalability.
- **Cons (async):** eventual consistency, duplicates, harder debugging.

### Alternatives
- **Hybrid:** sync for critical path + async for side effects.
- **How to choose:** default to sync for user-critical decisions, async for fan-out.

## Failure modes & Pitfalls
- Sync chains across many services → timeouts and cascading failures.
- Async pipelines without idempotency or DLQs → duplicate side effects and backlog.

## Observability (How to detect issues)
- **Metrics:** sync latency p95/p99, async lag, retry rate, DLQ depth.
- **Logs:** correlation IDs across sync and async flows.
- **Traces:** show where time is spent and queue gaps.
- **Alerts:** sustained lag or synchronous tail-latency spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Document decision per integration
  - [ ] Timeouts for sync calls
  - [ ] Idempotency + DLQ for async flows

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using async as a replacement for transactions.
  - **Why it’s bad:** adds eventual consistency without compensations.
  - **Better approach:** keep critical invariants in the sync path or use sagas.

## Interview readiness
### “Explain it like I’m teaching”
- Synchronous calls give immediate consistency but add coupling and failure propagation; asynchronous flows decouple and scale but require idempotency and observability.

### Trap questions (with answers)
1) **Q:** Is async always more scalable?
   - **A:** Not necessarily; it adds operational overhead and can bottleneck on consumers.
2) **Q:** Can async guarantee immediate consistency?
   - **A:** No; you trade consistency for decoupling unless you add coordination.
3) **Q:** Should all user-facing operations be async?
   - **A:** No; user-critical decisions usually need sync confirmation.

### Quick self-check (5 items)
- [ ] I can explain sync vs async trade-offs.
- [ ] I can choose per workflow.
- [ ] I can name an async failure mode.
- [ ] I can discuss idempotency needs.
- [ ] I can define observability metrics.

## Links (NO duplication)
### Prerequisites
- [Request-response architecture](../architecture/request-response.md)
- [Event-driven architecture](../architecture/event-driven-basics.md)

### Related topics
- [Message delivery semantics](../architecture/message-delivery-semantics.md)
- [Retries, exponential backoff, and jitter](../operations/retries-and-backoff.md)
- [DLQ & poison messages](../operations/dlq-and-poison-messages.md)

### Compare with
- [Request-response architecture](../architecture/request-response.md) — synchronous; async uses brokers/queues.

## Notes / Inbox (optional)
- N/A
