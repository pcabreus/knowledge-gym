---
id: archetype-concurrency-control
title: "Concurrency Control (Avoiding Double-Booking / Races)"
type: pattern
status: learning
importance: 95
difficulty: hard
tags: [system-design, locking, optimistic-locking, pessimistic-locking, concurrency, race-conditions]
canonical: true
aliases: [optimistic-locking, pessimistic-locking, leasing, compare-and-swap]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Concurrency Control (Avoiding Double-Booking / Races)

## TL;DR
- Guarantee correctness when multiple actors update the same resource (inventory, seats, wallets).
- Key challenges: overbooking, high contention on hot rows/keys, latency vs correctness tension.
- Solutions: optimistic concurrency (version field), pessimistic lock / lease, queueing per resource key.

## Where it hurts (why it hurts)
1. **Overbooking seats / inventory**: Two users buy last concert ticket simultaneously → both confirmed
   - **Solution**: Pessimistic lock (SELECT FOR UPDATE) or optimistic (version check + retry)
2. **High contention hot rows/keys**: Popular item → all requests serialize on lock → latency spike
   - **Solution**: Queue per resource; lease with expiry; sharding (if applicable)
3. **Latency vs correctness tension**: Strict locking → slow; optimistic → retries under contention
   - **Solution**: Choose based on contention: low → optimistic; high → pessimistic or queue

## Decision rules
- **Use when**: Scarce resources (inventory, seats, wallets), financial correctness critical, concurrent updates expected
- **Avoid when**: No concurrent updates (single writer), eventual consistency acceptable

## Trade-offs
- **Optimistic**: Fast (no locks), retry on conflict → good for low contention
- **Pessimistic**: Strict correctness, serialized → good for high contention, critical resources
- **Choose**: Low contention → optimistic; high contention / critical → pessimistic

## Explicit example
Two users try to buy the last concert ticket. Without concurrency control, both transactions read "1 available," both decrement to 0, both commit → overbooking. With pessimistic lock: first transaction locks row, second waits; first commits (sold out), second sees 0 available.

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [Optimistic Concurrency Control](../databases/optimistic-concurrency-control.md), [Lock-Based Concurrency](../databases/lock-based-concurrency-control.md), [Transactions](../databases/transactions.md)
