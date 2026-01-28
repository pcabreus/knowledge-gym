---
id: archetype-hot-partition
title: "Hot Partition / Celebrity Problem"
type: pattern
status: learning
importance: 85
difficulty: hard
tags: [system-design, hot-partition, skew, sharding, load-balancing]
canonical: true
aliases: [hotspotting, skew, key-imbalance, celebrity-problem]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Hot Partition / Celebrity Problem

## TL;DR
- A small subset of keys/entities dominate traffic, overloading one shard/partition.
- Key challenges: one partition melts while others idle, cascading failures (cache + DB + downstream).
- Solutions: key salting / fan-out replication, dedicated hot-key infrastructure, caching/CDN.

## Where it hurts (why it hurts)
1. **One partition melts while others idle**: Viral post gets 90% of reads → shard bottleneck
   - **Solution**: Detect skew early (monitor top keys); salt hot keys (replicate with `key:1`, `key:2`)
2. **Cascading failures**: Cache miss on hot key → DB overwhelmed → downstream timeouts
   - **Solution**: CDN / multi-layer caching; dedicated cache for hot keys
3. **Sharding doesn't help**: Adding more shards doesn't fix single-key hotspot
   - **Solution**: Replicate hot key across shards; approximate answers (sampling)

## Decision rules
- **Use when**: Social feeds, trending topics, live events, product pages with viral spikes
- **Avoid when**: Even traffic distribution (no celebrities)

## Trade-offs
- **Replication (salting)**: Spreads load but complicates invalidation
- **Dedicated infrastructure**: Fast but operational overhead
- **Approximate answers**: Cheap but less accurate

## Explicit example
Viral post gets 1M reads/sec. If sharded by post_id, all requests hit one shard → melts. Salt key: store 10 copies (`post:123:0` ... `post:123:9`); client reads random replica. Spreads load 10× across shards.

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [Distributed Caching](archetype-distributed-caching.md), [Partitioning & Sharding](partitioning-and-sharding.md), [DynamoDB Hot Partitions](../databases/dynamodb-hot-partitions.md)
