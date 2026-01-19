---
id: redis-data-types
title: "Redis data types"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [redis, data-structures, nosql]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Redis data types

## TL;DR (BLUF)
- Redis offers specialized data structures for different access patterns.
- Picking the right type reduces memory and improves atomic operations; sorted sets shine for ordered, ranked data.
- Trade-off: powerful primitives vs careful modeling of keys and memory.

## Definition
**What it is:** The core data structures Redis exposes (strings, hashes, lists, sets, sorted sets, [Redis Streams](redis-streams.md), bitmaps, hyperloglogs, and geospatial indexes).  
**Key terms:** strings, hashes, lists, sets, sorted sets (zsets), [Redis Streams](redis-streams.md), bitmaps, HyperLogLog, geo.

## Why it matters
- Data type choice affects memory use, latency, and correctness.
- It enables atomic operations without extra application logic.

## Scope & Non-goals
**In scope:** selecting data types to match access patterns and constraints.  
**Out of scope / NOT solved by this:** SQL-style querying or complex joins.

## Mental model / Intuition
- Treat Redis as a toolbox of specialized containers, not just a key-value map.
- **Sorted sets (zsets)** are a live leaderboard: items stay ordered as scores change.

## Decision rules (When to use / When not to use)
### Use it when
- You need atomic counters, sets, rankings, or queues at low latency.
- You want to avoid application-side locking or sorting.
- You need continuously ordered data (e.g., top-N, time-ordered items) without re-sorting on every read.
### Avoid it when
- You need multi-attribute queries across large datasets.
- You expect documents with evolving schemas and heavy analytics.
- You need global ordering across multiple keys or complex joins.

## How I would use it (practical)
- **Context:** Real-time leaderboards and per-user session data.
- **Steps:** use zsets for scores and rank queries → hashes for session fields → sets for membership.
- **What success looks like:** fast top-N queries without sorting in the app, predictable memory usage.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** atomic operations, low latency, purpose-built structures.
- **Cons / Risks:** memory overhead for small keys; modeling complexity; zset memory grows with member count.
### Alternatives
- **Document databases:** better for flexible documents and querying.
- **Relational databases:** better for complex joins and constraints.
- **How to choose:** use Redis data types when speed and atomicity matter more than query flexibility.

## Failure modes & Pitfalls
- Large values in strings causing memory pressure.
- Lists used as queues without trimming (unbounded growth).
- Using sets where ordering is required (should use sorted sets).
- Zsets without score normalization (e.g., mixing timestamps and ranks) leading to confusing ordering.
- Overusing zsets for multi-attribute ranking (should compute score or use a different store).

## Observability (How to detect issues)
- **Metrics:** memory usage by keyspace, `bigkeys` detection, latency p95.
- **Logs:** slowlog entries for heavy operations.
- **Alerts:** growth of large keys or memory fragmentation spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Map each access pattern to a data type.
  - [ ] Keep values small; prefer hashes for many fields.
   - [ ] Use zsets for ranking and time-series ordering.
  - [ ] Trim lists or streams to bound memory.
   - [ ] Define a consistent scoring function for zsets (e.g., rank, timestamp, or weighted score).

## Mini example (if applicable)
### Data types: when to use and examples
- **Strings (simple values)**
   - **Use when:** counters, flags, cached blobs, simple tokens.
   - **Avoid when:** you need field-level updates or partial reads.
   - **Example:** `SET session:123 "..."` and `INCR rate_limit:user:42`.

- **Hashes (fielded objects)**
   - **Use when:** many small fields per key (profiles, sessions).
   - **Avoid when:** you need complex queries across many hashes.
   - **Example:** `HSET user:42 name "Ana" plan "pro"`.

- **Lists (ordered, index-based, push/pop)**
   - **Use when:** stacks/queues with push/pop semantics.
   - **Avoid when:** you need ordering by score or time across many items.
   - **Example:** `LPUSH queue:emails id123` then `RPOP`.

- **Sets (unique, unordered membership)**
   - **Use when:** membership checks, de-dup, set ops.
   - **Avoid when:** you need ordering or ranking.
   - **Example:** `SADD feature:beta user:42` then `SISMEMBER`.

- **Sorted sets (zsets: ordered by score)**
   - **Use when:** leaderboards, time-ordered feeds, top-N queries.
   - **Avoid when:** you need multi-field sorting (compute a score or use another store).
   - **Example:** `ZADD leaderboard 9800 user:42` then `ZREVRANGE leaderboard 0 9 WITHSCORES`.

- **[Redis Streams](redis-streams.md) (append-only log)**
   - **Use when:** durable event processing with replay.
   - **Avoid when:** you only need fire-and-forget notifications.
   - **Example:** `XADD orders * id 123 status created`.

- **Bitmaps (bit operations over strings)**
   - **Use when:** large-scale flags or daily activity tracking.
   - **Avoid when:** you need sparse sets with variable IDs (use sets).
   - **Example:** `SETBIT active:2026-01-19 123 1`.

- **HyperLogLog (approximate cardinality)**
   - **Use when:** unique counts at huge scale with low memory.
   - **Avoid when:** you need exact counts.
   - **Example:** `PFADD uniques page:home user:42` then `PFCOUNT`.

- **Geospatial indexes (geo sets)**
   - **Use when:** radius queries and nearest neighbors.
   - **Avoid when:** you need complex spatial queries or polygons.
   - **Example:** `GEOADD stores -74.0 40.7 store:1` then `GEORADIUS`.

## Common anti-patterns
- **Anti-pattern:** Storing a full document blob in a string for frequent field updates.
  - **Why it’s bad:** requires rewrite of large values and wastes memory.
  - **Better approach:** use hashes for field-level updates.
- **Anti-pattern:** Using lists for leaderboards.
   - **Why it’s bad:** re-sorting on every read is expensive and error-prone.
   - **Better approach:** use sorted sets and keep ordering server-side.

## Interview readiness
### “Explain it like I’m teaching”
- Redis data types are specialized structures (hashes, sets, zsets, lists, etc.) that let you do atomic operations efficiently. Choosing the right type saves memory and simplifies logic.

### Trap questions (with answers)
1) **Q:** Are Redis hashes always more memory efficient than JSON strings?
   - **A:** not always; it depends on field count and encoding, but hashes enable field updates without rewriting the whole value.
2) **Q:** Is a list the best choice for a leaderboard?
   - **A:** no; sorted sets are designed for ranking.
3) **Q:** Can sets preserve ordering?
   - **A:** no; use sorted sets when order matters.
4) **Q:** Do sorted sets keep order automatically after updates?
   - **A:** yes; updating a member’s score reorders it automatically.

### Quick self-check (5 items)
- [ ] I can list common Redis data types and their use cases.
- [ ] I can pick between list vs stream vs zset.
- [ ] I can explain why hashes help with field updates.
- [ ] I can name at least one memory pitfall.
- [ ] I can match a data type to an access pattern.

## Links (NO duplication)
### Prerequisites
- [Redis](redis.md)
- [Key-value data modeling](key-value-data-modeling.md)

### Related topics
- [Caching fundamentals](../system-design/caching-fundamentals.md)
- [NoSQL access patterns](nosql-access-patterns.md)

### Compare with
- [Document databases (overview)](document-databases.md) — flexible schema and query vs in-memory structures.

## Notes / Inbox (optional)
- N/A
