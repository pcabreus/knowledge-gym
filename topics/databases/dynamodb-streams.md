---
id: dynamodb-streams
title: "DynamoDB Streams"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [database, dynamodb]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# DynamoDB Streams

## TL;DR (BLUF)
- DynamoDB Streams is a DynamoDB-native [Change Data Capture (CDC)](change-data-capture.md) mechanism.
- Use it for event-driven workflows and projections.
- Trade-off: at-least-once delivery and ordering constraints.

## Definition
**What it is:** A stream of change records (INSERT/MODIFY/REMOVE) for a DynamoDB table.
**Key terms:** CDC, stream records, at-least-once delivery.

## Why it matters
- It enables event-driven processing without polling.
- You must handle retries and duplicate events.

## Scope & Non-goals
**In scope:** change capture and downstream consumers.
**Out of scope / NOT solved by this:** exactly-once processing guarantees.

## Mental model / Intuition
- Think of Streams as a change log for your table.

## Decision rules (When to use / When not to use)
### Use it when
- You need to react to data changes.
- You want to build projections or trigger workflows.
### Avoid it when
- You need global ordering or exactly-once semantics.

## How I would use it (practical)
- **Context:** Syncing data to a search index.
- **Steps:** enable Streams → consume with Lambda → handle retries/idempotency.
- **What success looks like:** consistent downstream state.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** near-real-time change capture.
- **Cons / Risks:** duplicates and retries.
### Alternatives
- **Polling:** simpler but slower and more expensive.
- **How to choose:** use Streams when you need event-driven reactions.

## Failure modes & Pitfalls
- Missing idempotency in consumers.
- Backlog causing lag.

## Observability (How to detect issues)
- **Metrics:** stream lag, error rate, iterator age.
- **Logs:** consumer errors and retries.
- **Alerts:** rising iterator age.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Make consumers idempotent
  - [ ] Monitor iterator age

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Assuming exactly-once delivery.
  - **Why it’s bad:** leads to duplicates.
  - **Better approach:** make consumers idempotent.

## Interview readiness
### “Explain it like I’m teaching”
- DynamoDB Streams is a change log for table updates. It enables event-driven processing but delivers events at least once, so consumers must be idempotent.

### Trap questions (with answers)
1) **Q:** Are Streams events exactly once?
   - **A:** no; they’re at-least-once.
2) **Q:** Do Streams preserve global ordering?
   - **A:** no; ordering is per partition.
3) **Q:** Can TTL deletions appear in Streams?
   - **A:** yes; TTL deletions generate stream records.

### Quick self-check (5 items)
- [ ] I can define Streams precisely.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain ordering limits.

## Links (NO duplication)
### Prerequisites
- [DynamoDB](dynamodb.md)
- [Change Data Capture (CDC)](change-data-capture.md)

### Related topics
- [DynamoDB TTL](dynamodb-ttl.md)

### Compare with
- [Kafka](../architecture/kafka.md)
