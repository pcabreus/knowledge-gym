---
id: archetype-fan-out
title: "Fan-Out / Broadcast Delivery"
type: pattern
status: learning
importance: 85
difficulty: hard
tags: [system-design, fan-out, pub-sub, push-notifications, feeds]
canonical: true
aliases: [pub-sub-fan-out, push-notifications, feed-distribution]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Fan-Out / Broadcast Delivery

## TL;DR
- One event must be delivered to many consumers/targets (N followers, subscribers, devices).
- Key challenges: explosion of work (N targets), backpressure and retries, delivering "once" per target.
- Solutions: async queues + worker pools, topic-based pub/sub, fan-out on write vs read trade-off.

## Where it hurts (why it hurts)
1. **Explosion of work (N followers)**: One post → 5M deliveries → synchronous collapse
   - **Solution**: Async queues + worker pools for scalable delivery; batch where possible
2. **Backpressure and retries**: Failed deliveries must retry without blocking others
   - **Solution**: Per-target DLQ; retry with exponential backoff; parallelism
3. **Delivering "once" per target**: Retries cause duplicates
   - **Solution**: Idempotent delivery (track delivery per target); dedupe

## Decision rules
- **Use when**: Notifications, feeds, real-time updates, webhooks to partners
- **Avoid when**: Targets < 10 (direct calls simpler), strict real-time latency (fan-out adds delay)

## Trade-offs
- **Fan-out on write**: Precompute/deliver immediately (fast reads, slow/expensive writes)
- **Fan-out on read**: Compute on demand (fast writes, slow reads, cheaper storage)
- **Choose**: < 10k followers → fan-out on write; > 100k → fan-out on read

## Explicit example
Creator posts content; 5M followers should get notified. Synchronous fan-out collapses service. Async: publish event → worker pool fetches follower list in batches → enqueues 5M delivery tasks → workers process with retries.

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [High-Throughput Event Ingestion](archetype-event-ingestion.md), [Idempotency](archetype-idempotency-dedup.md)
