---
id: redis-eviction-policies
title: "Redis eviction policies"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [redis, cache, memory]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Redis eviction policies

## TL;DR (BLUF)
- Eviction policies decide which keys Redis removes when memory is full.
- Choose a policy that matches your cache strategy and TTL usage.
- Trade-off: predictability vs hit rate vs write availability.

## Definition
**What it is:** The configuration that tells Redis how to free memory when `maxmemory` is reached (e.g., LRU, LFU, random, TTL-based, or no eviction).  
**Key terms:** `maxmemory`, eviction, `allkeys-*`, `volatile-*`, LRU, LFU, noeviction.

## Why it matters
- The wrong policy can cause cascading cache misses or write failures.
- It directly affects latency, hit rate, and operational stability.

## Scope & Non-goals
**In scope:** memory pressure behavior, cache survivability, and write semantics.  
**Out of scope / NOT solved by this:** persistence durability, data modeling, or cache invalidation strategy.

## Mental model / Intuition
- A bouncer at a full club decides who gets kicked out to let new guests in.

## Decision rules (When to use / When not to use)
### Use it when
- You run Redis with bounded memory and need predictable behavior at capacity.
- You want to bias eviction toward stale or low-value data.
### Avoid it when
- You expect Redis to behave like a durable system of record.
- You rely on writes always succeeding (use `noeviction` only if your app can handle write failures).

## How I would use it (practical)
- **Context:** Cache-aside for API responses with TTLs.
- **Steps:** set `maxmemory` → choose policy → validate hit rate under load → alert on evictions.
- **What success looks like:** evictions only under load spikes and steady hit rate.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** controlled memory usage and predictable pressure behavior.
- **Cons / Risks:** evictions can drop hot keys or cause write failures.
### Alternatives
- **Scale up:** add memory to reduce eviction pressure.
- **Redis Cluster:** shard to distribute memory and hot keys.
- **How to choose:** if you can’t tolerate evictions, scale up or shard; otherwise pick a policy that matches TTL usage.

## Failure modes & Pitfalls
- `noeviction` causing write errors and downstream failures.
- `volatile-*` policies with missing TTLs (causes unexpected key retention).
- LRU/LFU on mixed workloads with huge values (evicting small hot keys hurts hit rate).

## Observability (How to detect issues)
- **Metrics:** `used_memory`, `maxmemory`, `evicted_keys`, `keyspace_hits/misses`, latency p95.
- **Logs:** slowlog entries around eviction spikes.
- **Alerts:** sustained evictions or sudden hit rate drops.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Set `maxmemory` explicitly (avoid default unbounded usage).
  - [ ] Choose `allkeys-lru` or `allkeys-lfu` for pure cache use cases.
  - [ ] Use `volatile-ttl` only if most keys have TTLs.
  - [ ] Load-test with realistic key sizes and access patterns.
- **Operational notes**
  - Evictions are normal under pressure; sustained evictions mean capacity mismatch.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using `volatile-*` without TTLs on most keys.
  - **Why it’s bad:** Redis won’t evict many keys, and memory pressure persists.
  - **Better approach:** set TTLs consistently or use `allkeys-*`.

## Interview readiness
### “Explain it like I’m teaching”
- Eviction policies define which keys Redis removes when memory is full. Pick a policy that matches your cache strategy and TTL usage to keep hit rates predictable.

### Trap questions (with answers)
1) **Q:** Does `noeviction` mean Redis is durable?
   - **A:** no; it only stops evictions and will fail writes instead.
2) **Q:** Is `volatile-lru` safe if you forget TTLs?
   - **A:** no; keys without TTLs won’t be evicted and can cause memory pressure.
3) **Q:** Do LRU/LFU always keep the hottest keys?
   - **A:** not perfectly; they are approximations and can be skewed by large values.

### Quick self-check (5 items)
- [ ] I can define what eviction policies do in Redis.
- [ ] I can choose between `allkeys-*` and `volatile-*`.
- [ ] I can explain why `noeviction` can break writes.
- [ ] I can name at least 2 eviction policies.
- [ ] I can describe how to monitor evictions.

## Links (NO duplication)
### Prerequisites
- [Redis](redis.md)
- [TTL (Time-to-Live)](ttl.md)
- [Caching fundamentals](../system-design/caching-fundamentals.md)

### Related topics
- [Caching strategy](../system-design/caching-strategy.md)
- [Hot partitions](hot-partitions.md)

### Compare with
- [Partitioning & sharding](../system-design/partitioning-and-sharding.md) — solves capacity by distributing data, not evicting it.

## Notes / Inbox (optional)
- N/A
