---
id: event-driven-basics
title: "Event-Driven Architecture"
type: concept
status: learning
importance: 60
difficulty: medium
tags: [architecture, messaging, distributed-systems]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-21
---

# Event-Driven Architecture

## TL;DR (BLUF)
- Event-driven architecture (EDA) publishes events and lets consumers react asynchronously.
- Use it to decouple services and scale workflows independently.
- Trade-off: eventual consistency, ordering/idempotency complexity, and harder debugging.

## Definition
**What it is:** An architecture where producers emit immutable events to a broker and consumers react asynchronously, often building their own local state or projections.
**Key terms:** event, producer, consumer, broker, topic/queue, event contract, idempotency.

## Why it matters
- It reduces coupling, enabling independent deploys and scalable fan-out.
- It shifts correctness challenges to ordering, duplication, and eventual consistency.

## Scope & Non-goals
**In scope:** event flows, contracts, reliability guarantees, and operational trade-offs.
**Out of scope / NOT solved by this:** broker internals, stream processing algorithms, or exact-once guarantees.

## Mental model / Intuition
- Think of a newsroom: publishers announce facts, subscribers decide what to do.
- Events are facts about state changes; consumers derive their own view from them.

## Decision rules (When to use / When not to use)
### Use it when
- You need asynchronous workflows with independent scaling.
- Multiple downstream systems should react to the same change.
- You can tolerate eventual consistency and design for idempotency.
### Avoid it when
- You need strict, immediate consistency across services.
- The workflow is simple and synchronous calls suffice.
- Your team lacks operational maturity for distributed debugging.

## How I would use it (practical)
- **Context:** An order triggers fulfillment, email, analytics, and fraud checks.
- **Steps:**
   1) Define an event contract (versioned schema).
   2) Publish `OrderCreated` to a broker.
   3) Consumers handle the event idempotently and store their own state.
   4) Add retries and a dead-letter queue for poison messages.
- **What success looks like:** independent scaling, no tight coupling, and clear operational visibility.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** loose coupling, scalable fan-out, better resilience to downstream failures.
- **Cons / Risks:** eventual consistency, ordering issues, and complex failure handling.
### Alternatives
- **[Request-response architecture](request-response.md):** simpler semantics for tight, synchronous workflows.
- **[Batch ETL](../operations/batch-etl.md):** better for periodic, large-scale data movement.
- **How to choose:** use EDA when you need async reactions and decoupled evolution.

## Failure modes & Pitfalls
- Duplicate events and out-of-order processing without idempotency.
- Schema drift between producers and consumers.
- Poison messages causing infinite retry loops.
- Silent backlogs due to under-provisioned consumers.

## Observability (How to detect issues)
- **Metrics:** consumer lag, throughput, retry rate, DLQ depth, processing latency.
- **Logs:** event ID, correlation ID, schema version, failure reason.
- **Traces:** producer → broker → consumer spans with timing gaps.
- **Alerts:** sustained lag growth, DLQ spikes, or high retry rates.

## Implementation notes (if applicable)
- **Checklist**
   - [ ] Define versioned event contracts
   - [ ] Make consumers idempotent
   - [ ] Use correlation IDs for tracing
   - [ ] Plan for retries and dead-letter queues
- **Security / Compliance notes**
   - Avoid putting sensitive data in events unless required and protected.
- **Performance notes**
   - Prefer small, immutable events; avoid large payloads.
- **Operational notes**
   - Monitor lag per consumer group and set SLOs.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using events as RPC.
   - **Why it’s bad:** adds latency and weakens consistency guarantees.
   - **Better approach:** use [Request-response architecture](request-response.md) for tight loops.
- **Anti-pattern:** Emitting internal DB rows as events without a contract.
   - **Why it’s bad:** brittle consumers and accidental coupling.
   - **Better approach:** publish explicit domain events with a stable schema.

## Interview readiness
### “Explain it like I’m teaching”
- Event-driven architecture publishes facts about changes as events, and independent consumers react asynchronously. It scales fan-out and reduces coupling, but it requires idempotency, ordering strategies, and strong observability.

### Trap questions (with answers)
1) **Q:** Are events globally ordered?
   - **A:** Usually no; ordering is typically per partition or per key.
2) **Q:** Does event-driven architecture guarantee exactly-once processing?
   - **A:** No; at-least-once is common, so consumers must be idempotent.
3) **Q:** Is EDA the same as event sourcing?
   - **A:** No; EDA is communication style, event sourcing is a persistence pattern.

### Quick self-check (5 items)
- [x] I can define event-driven architecture.
- [x] I can state when to use it.
- [x] I can name a trade-off.
- [x] I can describe a pitfall.
- [x] I can explain observability signals.

## Links (NO duplication)
### Prerequisites
- [Messaging basics](messaging-basics.md)
- [Distributed systems basics](../system-design/distributed-systems-basics.md)

### Related topics
- [Kafka](kafka.md)
- [Outbox pattern](outbox-pattern.md)
- [Dual-write pattern](dual-write-pattern.md)
- [Change Data Capture (CDC)](../databases/change-data-capture.md)

### Compare with
- [Request-response architecture](request-response.md) — synchronous coupling vs async reactions.
