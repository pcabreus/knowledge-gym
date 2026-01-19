---
id: rabbitmq-vs-kafka
title: "RabbitMQ vs Kafka"
type: concept
status: learning
importance: 40
difficulty: medium
tags: [architecture, messaging]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# RabbitMQ vs Kafka

## TL;DR (BLUF)
- RabbitMQ is a broker for queues; Kafka is a distributed log.
- Use RabbitMQ for task queues, Kafka for streaming and replay.
- Trade-off: Kafka adds operational complexity; RabbitMQ has less replay capability.

## Definition
**What it is:** A comparison of RabbitMQ (message broker) and Kafka (distributed log).
**Key terms:** queue, topic, partitions, acknowledgments.

## Why it matters
- Choosing the wrong broker leads to scaling or replay issues.

## Scope & Non-goals
**In scope:** conceptual differences and trade-offs.
**Out of scope / NOT solved by this:** deployment tuning.

## Mental model / Intuition
- RabbitMQ is a mailbox; Kafka is a durable log.

## Decision rules (When to use / When not to use)
### Use RabbitMQ when
- You need task queues and complex routing.
### Use Kafka when
- You need streaming, replay, and high throughput.

## How I would use it (practical)
- **Context:** Background job processing vs event streaming.
- **Steps:** map workload to broker strengths → validate throughput.
- **What success looks like:** stable throughput and manageable ops.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** RabbitMQ simpler for queues; Kafka excels at streaming.
- **Cons / Risks:** Kafka has higher operational cost.
### Alternatives
- **SQS/SNS:** managed queues.
- **How to choose:** use Kafka for streams, RabbitMQ for task queues.

## Failure modes & Pitfalls
- Using Kafka for simple task queues and overcomplicating.

## Observability (How to detect issues)
- **Metrics:** queue depth, consumer lag.
- **Logs:** broker errors.
- **Alerts:** growing backlog.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define workload type
  - [ ] Validate throughput needs

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Choosing Kafka for simplicity.
  - **Why it’s bad:** adds unnecessary ops complexity.
  - **Better approach:** use RabbitMQ or managed queues.

## Interview readiness
### “Explain it like I’m teaching”
- RabbitMQ is a traditional message broker great for queues and routing; Kafka is a distributed log built for streaming and replay. Pick based on workload.

### Trap questions (with answers)
1) **Q:** Is Kafka just a faster RabbitMQ?
   - **A:** no; they’re different models.
2) **Q:** Can RabbitMQ replay messages like Kafka?
   - **A:** not in the same way.
3) **Q:** Are consumer groups unique to Kafka?
   - **A:** they are core to Kafka’s model.

### Quick self-check (5 items)
- [ ] I can compare RabbitMQ and Kafka.
- [ ] I can state when to use each.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain replay differences.

## Links (NO duplication)
### Prerequisites
- [Messaging basics](messaging-basics.md)

### Related topics
- [Kafka](kafka.md)

### Compare with
- [Kafka](kafka.md) — log vs broker.
