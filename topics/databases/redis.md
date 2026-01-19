---
id: redis
title: "Redis"
type: technology
status: learning
importance: 65
difficulty: medium
tags: [database, cache, nosql]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Redis

## TL;DR (BLUF)
- In-memory key-value store with rich data structures and optional persistence.
- Use it for caching, rate limiting, queues, and low-latency lookups.
- Trade-off: memory cost and durability/consistency constraints.

## Definition
**What it is:** An in-memory data store that supports strings, hashes, lists, sets, and sorted sets with optional persistence.
**Key terms:** in-memory, TTL, persistence, eviction policies.

## Why it matters
- It enables very low-latency access for hot data and coordination patterns.
- Misuse can cause data loss or cache stampedes.

## Scope & Non-goals
**In scope:** caching, fast key-value access, lightweight queues.
**Out of scope / NOT solved by this:** complex relational queries or durable primary storage.

## Mental model / Intuition
- Think of Redis as a super-fast shared memory with data structures.

## Decision rules (When to use / When not to use)
### Use it when
- You need sub-millisecond access to hot data.
- You need TTL-based caching or lightweight counters/queues.
### Avoid it when
- You require strong durability as the system of record.
- Your dataset cannot fit in memory.

## How I would use it (practical)
- **Context:** API responses with repeated reads.
- **Steps:** add cache-aside → set TTLs → handle cache misses → monitor hit rate.
- **What success looks like:** high cache hit rate and reduced DB load.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** extremely fast, rich data structures.
- **Cons / Risks:** memory cost; persistence trade-offs.
### Alternatives
- **PostgreSQL:** durable relational storage.
- **DynamoDB:** durable key-value at scale.
- **How to choose:** use Redis for caching and fast ephemeral data; use durable DBs for source of truth.

## Failure modes & Pitfalls
- Evictions causing sudden cache misses.
- Thundering herd on cache expiry.

## Observability (How to detect issues)
- **Metrics:** hit ratio, memory usage, eviction count, latency p95.
- **Logs:** slowlog events.
- **Alerts:** rising evictions or hit ratio drops.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Choose eviction policy
  - [ ] Set TTLs
  - [ ] Monitor memory and hit ratio

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using Redis as the only system of record.
  - **Why it’s bad:** data loss on failure or misconfiguration.
  - **Better approach:** keep a durable database as source of truth.

## Interview readiness
### “Explain it like I’m teaching”
- Redis is an in-memory key-value store that’s great for caching and low-latency operations. It’s fast but not a replacement for a durable database.

### Trap questions (with answers)
1) **Q:** Is Redis a durable database by default?
   - **A:** no; persistence is optional and has trade-offs.
2) **Q:** Does Redis always improve performance?
   - **A:** no; poor cache strategy can add complexity without benefits.
3) **Q:** Can Redis store complex data types?
   - **A:** yes; it supports lists, sets, hashes, and more.

### Quick self-check (5 items)
- [ ] I can define Redis precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a caching example.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Caching fundamentals](../system-design/caching-fundamentals.md)

### Related topics
- [PostgreSQL](postgresql.md)
### Compare with
- [DynamoDB](dynamodb.md) — durable key-value vs in-memory cache.
- [PostgreSQL](postgresql.md) — durable relational storage.
