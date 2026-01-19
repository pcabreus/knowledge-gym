---
id: caching-strategy
title: "Caching Strategy"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [system-design, caching]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Caching Strategy

## TL;DR (BLUF)
- A strategy defines where, what, and how to cache.
- Use it to balance latency, freshness, and cost.
- Trade-off: more caching layers add complexity.

## Definition
**What it is:** The plan for caching layers, TTLs, invalidation, and ownership.
**Key terms:** cache-aside, write-through, invalidation.

## Why it matters
- Poor strategy leads to stale data or wasted cache.
- Good strategy reduces DB load and latency.

## Scope & Non-goals
**In scope:** strategic choices for caching.
**Out of scope / NOT solved by this:** low-level cache implementation details.

## Mental model / Intuition
- Caching is a product decision: what can be stale?

## Decision rules (When to use / When not to use)
### Use it when
- Multiple services depend on the same data.
### Avoid it when
- Correctness requires strictly fresh data.

## How I would use it (practical)
- **Context:** Product catalog with heavy reads.
- **Steps:** choose cache-aside → set TTLs → invalidate on updates.
- **What success looks like:** reduced DB load with acceptable staleness.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** lower latency and load.
- **Cons / Risks:** stale data and invalidation complexity.
### Alternatives
- **Denormalization:** precompute read models.
- **How to choose:** cache when read load dominates and staleness is acceptable.

## Failure modes & Pitfalls
- Cache stampedes and inconsistent invalidation.

## Observability (How to detect issues)
- **Metrics:** hit rate, stale read reports.
- **Logs:** cache misses.
- **Alerts:** hit rate drops or stale data spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define freshness requirements
  - [ ] Choose caching pattern
  - [ ] Monitor hit rate

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Caching everything without measuring.
  - **Why it’s bad:** complexity without benefit.
  - **Better approach:** cache only hot paths.

## Interview readiness
### “Explain it like I’m teaching”
- A caching strategy decides what to cache, how long, and how to invalidate. It’s a balance between freshness and performance.

### Trap questions (with answers)
1) **Q:** Should you always cache at every layer?
   - **A:** no; each layer adds complexity.
2) **Q:** Can you ignore invalidation if TTL is short?
   - **A:** not for correctness-critical data.
3) **Q:** Is cache hit rate the only metric?
   - **A:** no; stale data and miss penalties matter.

### Quick self-check (5 items)
- [ ] I can define caching strategy.
- [ ] I can state trade-offs.
- [ ] I can name a pitfall.
- [ ] I can describe a monitoring signal.
- [ ] I can compare with denormalization.

## Links (NO duplication)
### Prerequisites
- [Caching fundamentals](caching-fundamentals.md)

### Related topics
- [TTL (Time-to-Live)](../databases/ttl.md)

### Compare with
- [Denormalization](../databases/denormalization.md) — precompute vs cache.
