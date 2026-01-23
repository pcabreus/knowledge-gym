---
id: redis-streams
title: "Redis Streams"
type: technology
status: learning
importance: 55
difficulty: medium
tags: [redis, messaging, streams]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Redis Streams

## TL;DR (BLUF)
- Redis Streams provide an append-only log with consumer groups.
- Use them for durable event processing with retry and replay.
- Trade-off: more operational complexity than Pub/Sub.

## Definition
**What it is:** A Redis data type and feature that stores ordered messages with IDs and supports consumer groups for at-least-once processing.  
**Key terms:** stream, entry ID, consumer group, pending entries list (PEL), acknowledgements.

## Why it matters
- It provides durability and replay that Redis Pub/Sub lacks.
- It enables event-driven workflows without an external broker.

## Scope & Non-goals
**In scope:** durable messaging, replay, consumer groups, at-least-once processing.  
**Out of scope / NOT solved by this:** exactly-once delivery, complex routing, or cross-stream transactions.

## Mental model / Intuition
- Think of it as a lightweight commit log inside Redis.

## Decision rules (When to use / When not to use)
### Use it when
- You need persistent events with replay and consumer tracking.
- You want Redis-native queues without a separate broker.
### Avoid it when
- You need exactly-once semantics or multi-region durability.
- You need complex routing and [Backpressure](../system-design/backpressure.md) across many topics.

## How I would use it (practical)
- **Context:** Order events processed by multiple worker groups.
- **Steps:** `XADD` → `XGROUP CREATE` → consumers `XREADGROUP` → `XACK` and retry pending.
- **What success looks like:** stable processing lag with bounded pending entries.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** durability, replay, consumer groups.
- **Cons / Risks:** operational tuning, memory growth if not trimmed.
### Alternatives
- **Redis Pub/Sub:** ultra-low latency but no persistence.
- **Kafka:** stronger durability and ecosystem.
- **How to choose:** use Streams for Redis-native durability; use Kafka for large-scale streaming pipelines.

## Failure modes & Pitfalls
- Not trimming streams, leading to unbounded memory growth.
- Unacked pending entries that never get retried.
- Using streams without [Backpressure](../system-design/backpressure.md) controls.

## Observability (How to detect issues)
- **Metrics:** stream length, consumer lag, pending entries count, memory usage.
- **Logs:** failed consumer processing and retry counts.
- **Alerts:** growing pending list or lag beyond SLO.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define trimming strategy (`MAXLEN` or manual).
  - [ ] Monitor consumer lag and pending entries.
  - [ ] Implement retry and dead-letter handling.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Treating Streams like Pub/Sub without a retention policy.
  - **Why it’s bad:** memory grows without bounds.
  - **Better approach:** configure trimming and retention.

## Interview readiness
### “Explain it like I’m teaching”
- Redis Streams are an append-only log with consumer groups. Unlike Pub/Sub, they persist messages and let you replay or retry them.

### Trap questions (with answers)
1) **Q:** Are Redis Streams exactly once?
   - **A:** no; they are at-least-once and require idempotent consumers.
2) **Q:** Does Streams replace Kafka for all cases?
   - **A:** no; Kafka scales and persists better for large pipelines.
3) **Q:** Do Streams auto-trim by default?
   - **A:** no; you must configure trimming or manage retention.

### Quick self-check (5 items)
- [ ] I can explain Streams in 2–3 sentences.
- [ ] I can describe consumer groups and pending entries.
- [ ] I can set a retention/trimming strategy.
- [ ] I can explain at-least-once semantics.
- [ ] I can name when to choose Streams vs Pub/Sub.

## Links (NO duplication)
### Prerequisites
- [Redis](redis.md)
- [Redis data types](redis-data-types.md)
- [Messaging basics](../architecture/messaging-basics.md)

### Related topics
- [Redis Pub/Sub](redis-pub-sub.md)
- [Event-driven basics](../architecture/event-driven-basics.md)

### Compare with
- [Kafka](../architecture/kafka.md) — larger-scale durable log with stronger guarantees.

## Notes / Inbox (optional)
- N/A
