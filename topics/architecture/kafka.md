---
id: kafka
title: "Kafka (Core Concepts)"
type: technology
status: learning
importance: 50
difficulty: medium
tags: [architecture, messaging]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Kafka (Core Concepts)

## TL;DR (BLUF)
- Kafka is a distributed log for high-throughput event streaming.
- Use it for durable event pipelines and decoupled systems.
- Trade-off: operational complexity and eventual consistency.

## Definition
**What it is:** A distributed log-based messaging system with partitions and consumer groups.
**Key terms:** partition, offset, consumer group, retention.

## Why it matters
- It scales event processing and enables replay.
- Misconfiguration can cause lag and data loss.

## Scope & Non-goals
**In scope:** Kafka core concepts.
**Out of scope / NOT solved by this:** deep broker configuration.

## Mental model / Intuition
- Think of Kafka as a durable append-only log.

## Decision rules (When to use / When not to use)
### Use it when
- You need durable event streams and replay.
### Avoid it when
- You need simple queues with minimal ops.

## How I would use it (practical)
- **Context:** Event pipeline for analytics.
- **Steps:** define topics → partition by key → build consumers with offsets.
- **What success looks like:** low lag and reliable processing.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** scalable, durable, replayable.
- **Cons / Risks:** operational complexity and ordering limits.
### Alternatives
- **SQS/SNS:** simpler managed queues.
- **How to choose:** use Kafka for high-throughput streaming.

## Failure modes & Pitfalls
- Lag buildup from slow consumers.
- Poor partition keys causing hot partitions.

## Observability (How to detect issues)
- **Metrics:** consumer lag, throughput, error rate.
- **Logs:** broker errors.
- **Alerts:** rising lag or partition imbalance.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Choose good partition keys
  - [ ] Monitor consumer lag

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Treating Kafka as a queue without retention planning.
  - **Why it’s bad:** data loss risk.
  - **Better approach:** configure retention and replay strategy.

## Interview readiness
### “Explain it like I’m teaching”
- Kafka is a distributed event log. Producers append events to partitions, consumers read them using offsets, and you can replay events when needed.

### Trap questions (with answers)
1) **Q:** Does Kafka guarantee global ordering?
   - **A:** no; ordering is per partition.
2) **Q:** Is Kafka a database?
   - **A:** no; it’s a streaming log.
3) **Q:** Are consumers push-based?
   - **A:** no; they pull by offsets.

### Quick self-check (5 items)
- [ ] I can define Kafka.
- [ ] I can explain partitions and offsets.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain consumer lag.

## Links (NO duplication)
### Prerequisites
- [Event-driven basics](event-driven-basics.md)

### Related topics
- [Change Data Capture (CDC)](../databases/change-data-capture.md)

### Compare with
- [RabbitMQ vs Kafka](rabbitmq-vs-kafka.md)
