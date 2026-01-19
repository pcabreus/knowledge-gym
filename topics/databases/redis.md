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
- Trade-off: memory cost, durability options, and operational complexity at scale.

## Definition
**What it is:** An in-memory data store that supports [Redis data types](redis-data-types.md) and optional [Redis persistence (RDB/AOF)](redis-persistence.md).
**Key terms:** in-memory, [TTL (Time-to-Live)](ttl.md), [Redis persistence (RDB/AOF)](redis-persistence.md), [Redis eviction policies](redis-eviction-policies.md), [Redis replication & Sentinel](redis-replication-sentinel.md), [Redis scaling](redis-scaling.md), [Redis Pub/Sub](redis-pub-sub.md), [Redis Streams](redis-streams.md).

## Why it matters
- It enables very low-latency access for hot data and coordination patterns.
- Misuse can cause data loss or cache stampedes.

## Scope & Non-goals
**In scope:** caching, fast key-value access, counters, rate limiting, lightweight queues.
**Out of scope / NOT solved by this:** complex relational queries, analytics, or durable primary storage.

## Mental model / Intuition
- Think of Redis as a super-fast shared memory with data structures.

## Decision rules (When to use / When not to use)
### Use it when
- You need sub-millisecond access to hot data.
- You need TTL-based caching, counters, rate limiting, or short-lived queues.
- You can tolerate eventual persistence or accept Redis as a secondary store.
### Avoid it when
- You require strong durability as the system of record.
- Your dataset cannot fit in memory or you cannot accept eviction.
- You need complex joins or ad-hoc queries.

## How I would use it (practical)
- **Context:** API responses with repeated reads.
- **Steps:** add cache-aside → set TTLs → prevent stampede → monitor hit rate and evictions.
- **What success looks like:** high cache hit rate and reduced DB load.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** extremely fast, rich data structures, flexible primitives.
- **Cons / Risks:** memory cost; persistence trade-offs; clustering complexity.
### Alternatives
- **PostgreSQL:** durable relational storage.
- **DynamoDB:** durable key-value at scale.
- **How to choose:** use Redis for caching and fast ephemeral data; use durable DBs for source of truth.

## Failure modes & Pitfalls
- Evictions causing sudden cache misses.
- Thundering herd on cache expiry.
- Hot keys causing latency spikes.
- Overuse of large values causing memory pressure.
- Replication lag leading to stale reads if you read replicas.

## Observability (How to detect issues)
- **Metrics:** hit ratio, memory usage, fragmentation, eviction count, latency p95, replication lag.
- **Logs:** slowlog events, persistence errors.
- **Alerts:** rising evictions, hit ratio drops, or replication lag spikes.

## Implementation notes (if applicable)
- **Checklist**
   - [ ] Choose eviction policy (e.g., allkeys-lru)
   - [ ] Set TTLs on cache keys
   - [ ] Protect against stampede (lock or jittered TTLs)
   - [ ] Choose persistence mode (RDB vs AOF) if needed
   - [ ] Monitor memory, evictions, and hit ratio

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using Redis as the only system of record.
  - **Why it’s bad:** data loss on failure or misconfiguration.
  - **Better approach:** keep a durable database as source of truth.
- **Anti-pattern:** Using `KEYS *` in production.
   - **Why it’s bad:** blocks the server and spikes latency.
   - **Better approach:** use `SCAN` with pagination.

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
- [Redis eviction policies](redis-eviction-policies.md)
- [Redis data types](redis-data-types.md)
- [Redis Pub/Sub](redis-pub-sub.md)
- [Redis scaling (scale up and out)](redis-scaling.md)
- [Redis Streams](redis-streams.md)
- [Redis persistence (RDB/AOF)](redis-persistence.md)
- [Redis replication & Sentinel](redis-replication-sentinel.md)
### Compare with
- [DynamoDB](dynamodb.md) — durable key-value vs in-memory cache.
- [PostgreSQL](postgresql.md) — durable relational storage.
