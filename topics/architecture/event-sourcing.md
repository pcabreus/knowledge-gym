---
id: event-sourcing
title: "Event Sourcing"
type: pattern
status: learning
importance: 60
difficulty: hard
tags: [architecture, data, messaging]
canonical: true
aliases: []
created_at: 2026-01-21
updated_at: 2026-01-21
---

# Event Sourcing

## TL;DR (BLUF)
- Persist state as a sequence of immutable events and rebuild state from them.
- Use it when auditability, history, and complex workflows matter.
- Trade-off: higher complexity, eventual consistency, and operational overhead.

## Definition
**What it is:** A persistence pattern where all state changes are stored as append-only events in an event store, and current state is derived via projections. It is often paired with [Event-driven architecture](event-driven-basics.md).  
**Key terms:** event store, projection, snapshot, replay, append-only, schema evolution.

## Why it matters
- It provides a complete audit trail and enables time-travel debugging.
- It supports multiple read models without mutating historical truth.

## Scope & Non-goals
**In scope:** event storage, projection building, and replay semantics.  
**Out of scope / NOT solved by this:** real-time messaging reliability or broker operations.

## Mental model / Intuition
- Treat events like a bank ledger: you never overwrite history, you only append.
- State is a fold over the event stream.

## Decision rules (When to use / When not to use)
### Use it when
- You need a full audit trail or regulatory traceability.
- Business logic is complex and benefits from temporal analysis.
- You can tolerate eventual consistency in read models.
### Avoid it when
- CRUD simplicity is sufficient and history is not required.
- You cannot invest in projection infrastructure and versioning discipline.
- Low-latency synchronous reads of authoritative state are critical.

## How I would use it (practical)
- **Context:** A financial ledger that must support audits and historical queries.
- **Steps:**
  1) Model domain events and define event schemas.
  2) Persist events to an append-only event store.
  3) Build projections for query use-cases.
  4) Use snapshots to reduce replay time.
- **What success looks like:** fast rebuilds, accurate projections, and traceable history.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** complete history, easy auditing, flexible read models.
- **Cons / Risks:** complex infrastructure, projection lag, and schema evolution pain.
### Alternatives
- **CRUD + audit tables:** simpler but less flexible for time-travel queries.
- **[Change Data Capture (CDC)](../databases/change-data-capture.md):** captures DB changes but lacks explicit domain semantics.
- **How to choose:** use event sourcing when history and replay are first-class requirements.

## Failure modes & Pitfalls
- Projection drift when events are replayed with changed logic.
- Slow rebuilds without snapshots.
- Event schema evolution breaking old replays.
- Dual-write bugs if events and state are written separately.

## Observability (How to detect issues)
- **Metrics:** event store write latency, projection lag, replay duration, snapshot coverage.
- **Logs:** event version, projection errors, replay failures.
- **Traces:** event append → projection update spans.
- **Alerts:** growing projection lag or repeated replay failures.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define stable, versioned event schemas
  - [ ] Build idempotent projections
  - [ ] Add snapshots for large aggregates
- **Security / Compliance notes**
  - Encrypt sensitive event payloads at rest.
- **Performance notes**
  - Partition events by aggregate ID to keep replay efficient.
- **Operational notes**
  - Monitor projection lag per consumer.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Writing state first and emitting events later.
  - **Why it’s bad:** creates dual-write inconsistencies.
  - **Better approach:** treat events as the source of truth and derive state.

## Interview readiness
### “Explain it like I’m teaching”
- Event sourcing stores every change as an event and derives current state from those events. It gives perfect history but adds complexity around projections and schema evolution.

### Trap questions (with answers)
1) **Q:** Is event sourcing the same as event-driven architecture?
  - **A:** No; event sourcing is about persistence, [Event-driven architecture](event-driven-basics.md) is about communication.
2) **Q:** Can you rebuild state if you change event schemas?
   - **A:** Only if you maintain backward-compatible versions or migration logic.
3) **Q:** Do you still need idempotency in projections?
   - **A:** Yes; replays and retries can apply the same event multiple times.

### Quick self-check (5 items)
- [ ] I can define event sourcing precisely in 2–3 sentences.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Event-driven architecture](event-driven-basics.md)
- [Messaging basics](messaging-basics.md)
- [Data modeling basics](../databases/data-modeling-basics.md)
- (TODO) CQRS

### Related topics
- [Outbox pattern](outbox-pattern.md)
- [Dual-write pattern](dual-write-pattern.md)
- [Kafka](kafka.md)
- [Domain-Driven Design (DDD)](domain-driven-design.md)

### Compare with
- [Change Data Capture (CDC)](../databases/change-data-capture.md) — CDC captures DB changes; event sourcing models domain intent.

## Notes / Inbox (optional)
- N/A
