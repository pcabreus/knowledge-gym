---
title: Short-lived Caching
---

# Short-lived Caching

## Definition
Short-lived caching is a strategy for temporarily storing data for a brief period to reduce latency and load, especially for third-party or volatile data.

## Why it matters
It improves performance and reduces costs for high-latency or rate-limited sources, while minimizing staleness risk.

## When to use / when not to use
Use for data that changes frequently or is expensive to fetch, but where some staleness is acceptable. Not for critical, always-fresh data.

## Trade-offs
- + Reduces latency and external API calls
- + Can smooth out traffic spikes
- - Risk of serving stale data
- - Cache invalidation complexity

## Prerequisites
- [Caching fundamentals](caching-fundamentals.md)

## Related topics
- [API Gateway](api-gateway.md)
- [Webhooks](../operations/webhooks.md)

## Trap questions
1. What is a risk of short-lived caching?
   - Serving stale or outdated data to users.
2. When is short-lived caching most useful?
   - For third-party or volatile data where freshness is less critical.
3. How can you mitigate risks with short-lived caching?
   - Use appropriate TTLs and cache invalidation strategies.
