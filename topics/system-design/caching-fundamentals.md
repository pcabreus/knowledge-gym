---
id: caching-fundamentals
title: "Caching Fundamentals"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [system-design, caching]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Caching Fundamentals

## TL;DR (BLUF)
- Caching stores computed or fetched data to reduce latency and load.
- Use it when reads are frequent and data is relatively stable.
- Trade-off: cache invalidation and staleness risk.

## Definition
**What it is:** A technique to store and reuse data to speed up responses.
**Key terms:** cache-aside, TTL, invalidation.

## Why it matters
- It improves latency and reduces database load.
- Incorrect invalidation causes stale data bugs.

## Scope & Non-goals
**In scope:** caching basics and trade-offs.
**Out of scope / NOT solved by this:** full cache architecture patterns.

## Mental model / Intuition
- Cache is a shortcut; you must keep it reasonably fresh.

## Decision rules (When to use / When not to use)
### Use it when
- Read-heavy workloads dominate.
### Avoid it when
- Data changes frequently and correctness is critical.

## How I would use it (practical)
- **Context:** Read-heavy API endpoint.
- **Steps:** use cache-aside → set TTL → monitor hit rate.
- **What success looks like:** higher hit rate and lower DB load.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** lower latency, reduced DB load.
- **Cons / Risks:** stale data, invalidation complexity.
### Alternatives
- **Denormalization:** precompute reads.
- **How to choose:** cache when read load is high and tolerates slight staleness.

## Failure modes & Pitfalls
- Cache stampedes on expiration.

## Observability (How to detect issues)
- **Metrics:** hit rate, eviction count, latency.
- **Logs:** cache miss spikes.
- **Alerts:** sudden hit-rate drop.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Choose caching pattern
  - [ ] Set TTLs
  - [ ] Monitor hit rate

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Caching without invalidation strategy.
  - **Why it’s bad:** stale data.
  - **Better approach:** define TTL and invalidation rules.

## Interview readiness
### “Explain it like I’m teaching”
- Caching keeps recent or expensive-to-compute data close to the app. It speeds up reads but introduces staleness and invalidation challenges.

### Trap questions (with answers)
1) **Q:** Is caching always safe?
   - **A:** no; you can serve stale data.
2) **Q:** Can TTL replace invalidation?
   - **A:** sometimes, but not for strict correctness.
3) **Q:** Are cache misses always bad?
   - **A:** no; they’re expected and should be handled.

### Quick self-check (5 items)
- [ ] I can define caching.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain a monitoring signal.

## Links (NO duplication)
### Prerequisites
- [Performance basics](performance-basics.md)

### Related topics
- [TTL (Time-to-Live)](../databases/ttl.md)

### Compare with
- [Caching strategy](caching-strategy.md) — patterns and choices.
