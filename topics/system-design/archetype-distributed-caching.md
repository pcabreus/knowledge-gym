---
id: archetype-distributed-caching
title: "Distributed Caching (Read Hotspots)"
type: pattern
status: learning
importance: 95
difficulty: medium
tags: [system-design, caching, cdn, cache-aside, read-through, stampede]
canonical: true
aliases: [cache-aside, read-through, write-through, cdn-caching]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Distributed Caching (Read Hotspots)

## TL;DR (BLUF)
- Use cache layers to reduce latency and offload storage/computation for read-heavy workloads.
- Key challenge: cache invalidation, staleness, cache stampede, hot keys dominating traffic.
- Solution patterns: cache-aside + TTL, stale-while-revalidate, request coalescing (singleflight).

## Definition
**What it is:** A system archetype that places fast, temporary storage (cache) between clients and authoritative data sources to reduce latency, cost, and load.

**Key terms:**
- **Cache-aside**: Application checks cache first; on miss, loads from source and populates cache
- **Read-through**: Cache loads from source automatically on miss
- **Write-through**: Writes go through cache to source
- **TTL (Time To Live)**: How long cached data remains valid
- **Cache stampede**: Many requests miss cache simultaneously, overwhelming source
- **Hot key**: A small set of keys receiving disproportionate traffic

## Why it matters
- **Latency**: Cache hits are 10-100× faster than source (ms vs µs)
- **Cost**: Reduces expensive DB queries, API calls, computation
- **Scalability**: Offloads read load from bottleneck sources
- **Interview frequency**: Appears in almost every system design interview

**Typical real-world consequences of getting it wrong:**
- Product page: Influencer post causes traffic spike → cache misses → DB CPU spikes → p99 latency explosion → site outage
- API rate limits: No caching of third-party API → hit rate limits → service degraded
- Incorrect invalidation: Stale data shown to users → customer support issues, revenue loss

## Scope & Non-goals
**In scope:**
- Cache patterns (aside, read-through, write-through)
- Cache invalidation strategies
- Handling cache stampede
- Hot key management

**Out of scope / NOT solved by this:**
- Distributed consistency → see [Multi-Region Replication](archetype-multi-region.md)
- Session storage → separate concern (sticky sessions, Redis)
- Write-heavy workloads → caching doesn't help

## Mental model / Intuition
Think of it like a **library reference desk**:
- Full library (source): Slow to search, comprehensive
- Reference desk (cache): Fast, commonly requested books only
- Librarian (cache policy): Knows which books to keep at desk (hot keys), when to return to shelves (TTL)
- Rush hour (stampede): Everyone asks for the same book at once; librarian fetches once, serves all

## Decision rules (When to use / When not to use)
### Use it when
- **Read-heavy**: Reads >> writes (10:1 or more)
- **Read patterns predictable**: Same data requested repeatedly
- **Latency critical**: p99 latency SLO tight
- **Expensive source**: DB queries slow, API calls rate-limited

### Avoid it when
- **Write-heavy**: Writes >> reads (cache constantly invalidated)
- **Unique reads**: Every request different (low hit rate)
- **Strong consistency required**: Can't tolerate staleness
- **Simple workload**: Premature optimization

## How I would use it (practical)
### Context: E-commerce product page
**Problem:** Product pages get 10k req/sec. DB query is 50ms p99. Need p99 < 200ms total. Influencer posts cause 100× spikes on specific products.

**Steps:**
1. **Cache layer**: Redis (multi-tier: CDN → Redis → DB)
2. **Pattern**: Cache-aside
   - Check Redis for `product:{id}`
   - On hit: return (< 5ms)
   - On miss: query DB, store in Redis with TTL=60s, return
3. **TTL strategy**: 60s for product data (balances freshness vs hit rate)
4. **Stampede protection**: Singleflight (one request fetches, others wait)
5. **Hot key handling**: For viral products, extend TTL to 5m during spike; use stale-while-revalidate
6. **Invalidation**: On product update, delete cache key (or set TTL=0)

**What success looks like:**
- Cache hit rate > 95%
- p99 latency < 100ms (mostly cache hits)
- Stampede: DB sees 1 query per key per TTL period (not 10k)

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Massive latency reduction (10-100×)
  - Reduces load on expensive sources
  - Horizontal scaling (add more cache nodes)
- **Cons / Risks:**
  - Cache invalidation is hard (stale data)
  - Cache stampede on misses (thundering herd)
  - Hot keys can overwhelm single cache node
  - Operational complexity (cache layer to manage)

### Alternatives
- **No caching (read from source):** Simple but slow and expensive
  - **When better:** Low traffic, strong consistency required
- **Materialized view**: Precompute instead of cache on-demand
  - **When better:** Complex aggregations, known query patterns
- **CDN only**: Edge caching for static/semi-static content
  - **When better:** Content rarely changes, geographically distributed users

**How to choose:**
- Start with CDN for static assets
- Add Redis cache for dynamic data with predictable reads
- Add materialized views for complex queries

## Failure modes & Pitfalls
### Where it hurts (why it hurts)
1. **Cache invalidation (classic hard problem)**
   - **Symptom:** Users see stale data (old prices, inventory, profile)
   - **Why:** Update in source not reflected in cache; invalidation missed
   - **Solution:** Short TTLs for changing data; explicit delete on write; version keys

2. **Stale data vs consistency**
   - **Symptom:** Users report "bug" when seeing outdated info
   - **Why:** Cache serves old version; source has new version
   - **Solution:** Set TTL based on staleness tolerance; critical reads bypass cache

3. **Cache stampede (thundering herd)**
   - **Symptom:** Cache key expires; 10k requests hit DB simultaneously; DB melts
   - **Why:** Many requests miss at same time (TTL expiry or cold start)
   - **Solution:** Singleflight (request coalescing), stale-while-revalidate

4. **Hot keys (one item dominates traffic)**
   - **Symptom:** Single cache node overloaded; other nodes idle
   - **Why:** Key hashing sends all requests for hot item to one node
   - **Solution:** Replicate hot key across nodes (prefix with random), CDN for static content

## Observability (How to detect issues)
### Metrics
- **Cache hit rate** (hits / (hits + misses)): Target > 90%
- **Cache latency p99**: Should be < 10ms
- **Stampede events**: Concurrent misses for same key
- **Hot key detection**: Top keys by request count

### Logs
- Log miss + source fetch: key, fetch_duration, outcome

### Traces
- Span: `check_cache` → (miss) → `fetch_source` → `populate_cache`

### Alerts
- `cache_hit_rate < 80% for 5 minutes` → TTL too short or invalidation too aggressive
- `cache_stampede_events > 10/min` → add singleflight protection
- `hot_key_requests > 10k/sec` → replicate hot key

## Implementation notes
### Checklist
- [ ] Choose cache technology (Redis, Memcached, CDN)
- [ ] Define TTL per data type (balance staleness vs freshness)
- [ ] Implement cache-aside pattern with fallback to source
- [ ] Add stampede protection (singleflight or lock)
- [ ] Monitor hit rate and latency
- [ ] Plan invalidation strategy (TTL, explicit delete, versioned keys)
- [ ] Handle hot keys (replication, CDN)

### Performance notes
- **Hit rate**: 90%+ is good; < 70% means caching may not be helping
- **TTL tuning**: Shorter TTL = fresher data but more misses; start with 60s and tune
- **Key size**: Smaller keys = more fit in memory; use compressed formats

### Operational notes
- **Eviction policy**: LRU (Least Recently Used) most common; tune based on workload
- **Memory sizing**: Monitor memory usage; ensure headroom to avoid evictions of hot data

## Mini example (code)
```python
import redis
from threading import Lock

cache = redis.Redis(host='localhost', port=6379)
fetch_locks = {}  # Singleflight: one fetch per key

def get_product(product_id):
    key = f"product:{product_id}"
    
    # Cache-aside: check cache first
    cached = cache.get(key)
    if cached:
        return json.loads(cached)
    
    # Singleflight: only one request fetches
    lock = fetch_locks.setdefault(product_id, Lock())
    with lock:
        # Double-check after acquiring lock
        cached = cache.get(key)
        if cached:
            return json.loads(cached)
        
        # Fetch from source
        product = db.query("SELECT * FROM products WHERE id = ?", product_id)
        
        # Populate cache with TTL
        cache.setex(key, 60, json.dumps(product))  # 60s TTL
        return product

# Invalidation on write
def update_product(product_id, updates):
    db.update("products", product_id, updates)
    cache.delete(f"product:{product_id}")  # Explicit invalidation
```

## Common anti-patterns
### Anti-pattern: No TTL (cache forever)
- **What people do:** Set cache without expiration
- **Why it's bad:** Stale data persists indefinitely; manual invalidation error-prone
- **Better approach:** Always set TTL; short for changing data, long for stable data

### Anti-pattern: No stampede protection
- **What people do:** Every request on miss queries source independently
- **Why it's bad:** Cache expiry causes thundering herd → DB overload
- **Better approach:** Singleflight (one fetch per key) or stale-while-revalidate

### Anti-pattern: Cache everything
- **What people do:** Cache every query result, regardless of access pattern
- **Why it's bad:** Memory waste on rarely-accessed data; low hit rate
- **Better approach:** Cache only hot keys; use metrics to identify what to cache

### Anti-pattern: Ignoring hot keys
- **What people do:** Assume even distribution; one cache node handles viral content
- **Why it's bad:** Bottleneck on one node; others idle
- **Better approach:** Detect hot keys; replicate across nodes or use CDN

## Interview readiness
### "Explain it like I'm teaching"
Caching is about putting a fast, temporary copy of data close to where it's needed. We use cache-aside: check the cache first; if it's there (a hit), return it immediately. If not (a miss), fetch from the slow source, store it in cache, and return it. We set a TTL so the cache doesn't serve stale data forever. The hard parts are: (1) cache stampede—when the cache expires and everyone hits the source at once, and (2) hot keys—when one popular item overloads a single cache node. We solve these with request coalescing and key replication.

### Trap questions (with answers)
1) **Q:** What's the difference between cache-aside and read-through?
   - **A:** Cache-aside: Application code checks cache, fetches from source on miss, and populates cache. Read-through: Cache layer itself fetches from source on miss (transparent to app). Cache-aside gives more control; read-through is simpler but less flexible.

2) **Q:** How do you invalidate cache on writes?
   - **A:** Three options: (1) Delete the cache key on write (lazy; next read repopulates). (2) Update the cache on write (eager; risk of stale if write fails). (3) Use TTL only (simple but may serve stale until expiry). Most systems use (1) + TTL as safety net.

3) **Q:** What is cache stampede and how do you prevent it?
   - **A:** Stampede: Cache key expires; many requests miss simultaneously and hit the source, overloading it. Prevent with singleflight (request coalescing): when a miss occurs, one request fetches while others wait for the result. Alternative: stale-while-revalidate (serve stale data, refresh in background).

4) **Q:** What's a hot key and how do you handle it?
   - **A:** Hot key: One key (e.g., viral post) receives disproportionate traffic. Hashing sends all requests to one cache node → bottleneck. Solutions: (1) Replicate key with random prefix (e.g., `post:123:1`, `post:123:2`); client picks random replica. (2) Use CDN for static content. (3) Increase cache node capacity.

5) **Q:** Why not just add more database servers instead of caching?
   - **A:** Caching is 10-100× faster (µs vs ms) and cheaper (memory vs DB instances). Also, many workloads are read-heavy; scaling DBs helps writes but not read latency. Caching is usually the first step; scale DBs if writes are the bottleneck.

## Links
**Part of:**
- [System Design Archetypes](system-design-archetypes.md)

**Prerequisites:**
- [Caching Fundamentals](caching-fundamentals.md)
- [Caching Strategy](caching-strategy.md)

**Related archetypes:**
- [Hot Partition / Celebrity Problem](archetype-hot-partition.md) — related to hot key problem
- [Rate Limiting & Quotas](archetype-rate-limiting-quotas.md) — protect cache from abuse

**Related topics:**
- [Redis](../databases/redis.md)
- [CDN (Content Delivery Network)](../operations/cdn.md) (if exists)
