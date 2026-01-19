---
id: redis-pub-sub
title: "Redis Pub/Sub"
type: pattern
status: learning
importance: 50
difficulty: medium
tags: [redis, messaging, pubsub]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Redis Pub/Sub

## TL;DR (BLUF)
- Redis Pub/Sub broadcasts messages to currently connected subscribers.
- It’s ultra-low latency but has no persistence or delivery guarantees.
- Trade-off: simplicity and speed vs reliability and replay.

## Definition
**What it is:** A Redis feature that delivers published messages to active subscribers on channels or patterns, without storing them.  
**Key terms:** channels, patterns, fan-out, at-most-once delivery.

## Why it matters
- It’s a fast way to push events but can silently drop messages.
- Misuse leads to missing updates and data inconsistency.

## Scope & Non-goals
**In scope:** real-time notifications, best-effort fan-out, transient messaging.  
**Out of scope / NOT solved by this:** durable queues, retries, or replay.

## Mental model / Intuition
- It’s a live radio broadcast: you only hear what’s on while you’re tuned in.

## Decision rules (When to use / When not to use)
### Use it when
- You need low-latency fan-out and can tolerate occasional loss.
- Consumers are online and can recompute state if they miss events.
### Avoid it when
- You need durability, ordering guarantees, or replay.
- Consumers must process every message exactly once.

## How I would use it (practical)
- **Context:** Live UI notifications for connected clients.
- **Steps:** publish events → clients subscribe while connected → fall back to polling on reconnect.
- **What success looks like:** users see near-real-time updates with acceptable occasional gaps.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** extremely fast, simple topology.
- **Cons / Risks:** no persistence, no acknowledgements, no replay.
### Alternatives
- **[Redis Streams](redis-streams.md):** add persistence and consumer groups.
- **Kafka:** durable log with replay and ordering.
- **How to choose:** if you need durability, use streams or a message broker.

## Failure modes & Pitfalls
- Subscriber disconnects cause silent message loss.
- Burst traffic overwhelms slow clients.
- Overloading a single channel with unrelated events.

## Observability (How to detect issues)
- **Metrics:** subscriber count, publish rate, client latency.
- **Logs:** client disconnects and reconnects.
- **Alerts:** sudden drop in subscribers or spikes in publish rate.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Ensure clients handle reconnects and state resync.
  - [ ] Separate channels by domain to reduce noise.
  - [ ] Add idempotency in consumers if possible.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using Pub/Sub for durable job processing.
  - **Why it’s bad:** messages are lost on disconnect or overload.
  - **Better approach:** use Redis Streams or a queue.

## Interview readiness
### “Explain it like I’m teaching”
- Redis Pub/Sub is a fire-and-forget broadcast system. It’s fast but doesn’t store messages, so subscribers only get events while connected.

### Trap questions (with answers)
1) **Q:** Does Redis Pub/Sub guarantee delivery?
   - **A:** no; it’s at-most-once delivery to active subscribers.
2) **Q:** Can you replay missed messages?
   - **A:** no; use Redis Streams or Kafka for replay.
3) **Q:** Is Pub/Sub good for background jobs?
   - **A:** no; it lacks durability and acknowledgements.

### Quick self-check (5 items)
- [ ] I can explain Redis Pub/Sub in one minute.
- [ ] I can state when it is appropriate vs not.
- [ ] I can name one durability alternative.
- [ ] I can describe the failure mode on disconnect.
- [ ] I can design a reconnect/resync strategy.

## Links (NO duplication)
### Prerequisites
- [Redis](redis.md)
- [Messaging basics](../architecture/messaging-basics.md)

### Related topics
- [Event-driven basics](../architecture/event-driven-basics.md)

### Compare with
- [Kafka](../architecture/kafka.md) — durable log with replay vs transient broadcast.
- [RabbitMQ vs Kafka](../architecture/rabbitmq-vs-kafka.md) — broker choices with delivery guarantees.

## Notes / Inbox (optional)
- N/A
