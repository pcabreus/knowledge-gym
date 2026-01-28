---
id: archetype-ordering-guarantees
title: "Ordering Guarantees (Per-Key Sequencing)"
type: pattern
status: learning
importance: 90
difficulty: hard
tags: [system-design, ordering, fifo, sequencing, event-ordering]
canonical: true
aliases: [ordered-delivery, fifo-per-key, sequencing-service]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Ordering Guarantees (Per-Key Sequencing)

## TL;DR
- Ensure events/commands for the same entity are processed in order.
- Key challenges: reordering causes invalid state transitions, parallelism vs order trade-off.
- Solutions: partitioned logs (Kafka partitions by key), FIFO queues per entity, state machine validation.

## Where it hurts (why it hurts)
1. **Reordering causes invalid state transitions**: OrderPaid processed after OrderRefunded → wrong state
   - **Solution**: Partition by entity ID (order_id) → all events for one order in-order
2. **Parallelism vs order trade-off**: Global ordering kills parallelism; no ordering → bugs
   - **Solution**: "Ordering per key" (not global) → parallel across keys, ordered within key
3. **Replays and dedupe must preserve correctness**: Replay reorders events → state corruption
   - **Solution**: Idempotent + state machine validation (reject invalid transitions)

## Decision rules
- **Use when**: State machines (order lifecycle, chat), finance ledgers, per-user/per-session ordering critical
- **Avoid when**: No state dependencies (events independent), global ordering not needed

## Trade-offs
- **Per-key ordering**: Balance parallelism + correctness; most practical
- **Global ordering**: Simple but no parallelism; rarely needed
- **No ordering**: Max parallelism but requires idempotent + commutative operations

## Explicit example
Order events: OrderPlaced → OrderPaid → OrderShipped. If reordered (OrderShipped before OrderPaid), state machine breaks. Partition by order_id ensures all events for one order go to same partition → ordered processing. Different orders go to different partitions → parallelism.

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [High-Throughput Event Ingestion](archetype-event-ingestion.md), [Kafka](../architecture/kafka.md), [Message Delivery Semantics](../architecture/message-delivery-semantics.md)
