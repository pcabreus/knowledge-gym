---
id: change-data-capture
title: "Change Data Capture (CDC)"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [database, streaming]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Change Data Capture (CDC)

## TL;DR (BLUF)
- CDC captures data changes and emits them to downstream systems.
- Use it for projections, sync, and event-driven workflows.
- Trade-off: at-least-once delivery and ordering limits.

## Definition
**What it is:** A mechanism to stream inserts/updates/deletes from a database to consumers.
**Key terms:** log-based CDC, change stream, ordering.

## Why it matters
- It enables near-real-time integration without polling.
- It requires idempotent consumers and replay handling.

## Scope & Non-goals
**In scope:** CDC concepts and downstream consumption.
**Out of scope / NOT solved by this:** exactly-once guarantees across systems.

## Mental model / Intuition
- Think of CDC as a change log feeding subscribers.

## Decision rules (When to use / When not to use)
### Use it when
- You need to propagate data changes to other systems.
### Avoid it when
- A batch sync is sufficient and simpler.

## How I would use it (practical)
- **Context:** Syncing DB changes to a search index.
- **Steps:** enable CDC → consume stream → ensure idempotency → monitor lag.
- **What success looks like:** consistent downstream state with low lag.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** near-real-time sync.
- **Cons / Risks:** duplicates, ordering constraints, lag management.
### Alternatives
- **Polling:** simpler but slower.
- **How to choose:** use CDC when timeliness matters and you can handle duplicates.

## Failure modes & Pitfalls
- Consumers not idempotent.
- Backlog and lag spikes.

## Observability (How to detect issues)
- **Metrics:** lag, error rate, retry rate.
- **Logs:** consumer errors.
- **Alerts:** rising lag or retry spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Make consumers idempotent
  - [ ] Monitor lag

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Assuming exactly-once delivery.
  - **Why it’s bad:** duplicates are normal.
  - **Better approach:** design idempotent consumers.

## Interview readiness
### “Explain it like I’m teaching”
- CDC streams database changes to other systems. It’s powerful for real-time sync, but you must handle duplicates and lag.

### Trap questions (with answers)
1) **Q:** Is CDC always exactly-once?
   - **A:** no; most CDC systems are at-least-once.
2) **Q:** Is CDC always better than polling?
   - **A:** no; polling can be simpler for low-frequency use.
3) **Q:** Can you ignore ordering?
   - **A:** no; ordering matters for correctness.

### Quick self-check (5 items)
- [ ] I can define CDC.
- [ ] I can explain when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a failure mode.
- [ ] I can describe a monitoring signal.

## Links (NO duplication)
### Prerequisites
- [Event-driven basics](../architecture/event-driven-basics.md)

### Related topics
- [DynamoDB Streams](dynamodb-streams.md)

### Compare with
- [Batch ETL](../operations/batch-etl.md)
