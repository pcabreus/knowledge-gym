---
id: outbox-pattern
title: "Outbox Pattern"
type: pattern
status: learning
importance: 50
difficulty: medium
tags: [architecture, messaging]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Outbox Pattern

## TL;DR (BLUF)
- The outbox pattern writes events to a table in the same transaction as business data.
- Use it to ensure reliable event publication.
- Trade-off: extra storage and processing complexity.

## Definition
**What it is:** A transactional pattern that persists events in an outbox table and later publishes them.
**Key terms:** outbox table, CDC, idempotency.

## Why it matters
- It prevents lost events when transactions commit but publish fails.
- It enables reliable event-driven workflows.

## Scope & Non-goals
**In scope:** reliable event publishing from a DB.
**Out of scope / NOT solved by this:** exactly-once delivery.

## Mental model / Intuition
- Write your intent to the database first, then publish safely.

## Decision rules (When to use / When not to use)
### Use it when
- You need transactional consistency between DB writes and events.
### Avoid it when
- You can tolerate occasional missed events.

## How I would use it (practical)
- **Context:** Order created event.
- **Steps:** write order + outbox row → CDC/worker publishes → mark as sent.
- **What success looks like:** no lost events under failures.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** reliable event publishing.
- **Cons / Risks:** extra processing and storage.
### Alternatives
- **Dual writes:** simpler but risky.
- **How to choose:** use outbox when reliability is required.

## Failure modes & Pitfalls
- Outbox table growth without cleanup.

## Observability (How to detect issues)
- **Metrics:** outbox lag, publish error rate.
- **Logs:** publish failures.
- **Alerts:** growing outbox backlog.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Write outbox in same transaction
  - [ ] Publish with retries and idempotency

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Publishing directly before commit.
  - **Why it’s bad:** lost events on rollback.
  - **Better approach:** use an outbox table.

## Interview readiness
### “Explain it like I’m teaching”
- The outbox pattern writes events to a DB table in the same transaction as your data, then publishes them asynchronously. This avoids losing events if publishing fails.

### Trap questions (with answers)
1) **Q:** Does outbox guarantee exactly-once delivery?
   - **A:** no; you still need idempotent consumers.
2) **Q:** Can you skip cleanup?
   - **A:** no; outbox tables can grow indefinitely.
3) **Q:** Is outbox only for Kafka?
   - **A:** no; it works with any event transport.

### Quick self-check (5 items)
- [ ] I can define the outbox pattern.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a failure mode.
- [ ] I can explain how CDC helps.

## Links (NO duplication)
### Prerequisites
- [Transactions](../databases/transactions.md)

### Related topics
- [Change Data Capture (CDC)](../databases/change-data-capture.md)

### Compare with
- [Dual-write pattern](dual-write-pattern.md)
