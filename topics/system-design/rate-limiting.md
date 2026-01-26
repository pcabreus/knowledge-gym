---
id: rate-limiting
title: "Rate Limiting"
type: pattern
status: learning
importance: 75
difficulty: medium
tags: [system-design, reliability, security, api-design]
canonical: true
aliases: [throttling, quota]
created_at: 2026-01-26
updated_at: 2026-01-26
---

# Rate Limiting

## TL;DR (BLUF)
- Controls the rate of requests per client/user/IP/endpoint to prevent abuse and overload.
- Use it for public APIs, cost control, and protecting shared resources.
- Trade-off: legitimate users may be throttled during bursts.

## Definition
**What it is:** A control mechanism that limits the number of requests a client can make within a time window (e.g., 100 requests per minute), rejecting excess requests with 429 (Too Many Requests).

**Key terms:** quota, throttling, token bucket, leaky bucket, sliding window, fixed window, rate limit, backpressure.

## Why it matters
- Protects systems from abuse (DDoS, scrapers, runaway scripts).
- Ensures fair resource allocation across users/tenants.
- Controls infrastructure costs (API calls, DB queries, third-party fees).
- Complements [Backpressure](backpressure.md) and [Load shedding](../operations/load-shedding.md).

## Scope & Non-goals
**In scope:** per-user, per-IP, per-API-key, per-endpoint quotas; algorithms (token bucket, leaky bucket, sliding window).

**Out of scope / NOT solved by this:**
- Protecting against *already accepted* slow requests (use [Timeout](../operations/timeouts.md))
- Internal service-to-service rate limiting (use [Bulkhead](../operations/bulkheads.md) or [Adaptive concurrency](../operations/adaptive-concurrency.md))
- Application-level logic errors (use circuit breakers and retries)

## Mental model / Intuition
- Like a nightclub bouncer: only X people per hour. If the quota is full, you wait or get rejected.
- In APIs: each client has a "token budget" that refills over time. Requests consume tokens.

## Decision rules (When to use / When not to use)
### Use it when
- You have a public API or multi-tenant system.
- You need to prevent abuse, scraping, or runaway automation.
- Infrastructure costs scale with usage (pay-per-request third parties, cloud APIs).
- You want to enforce SLA tiers (free: 100 req/min, premium: 10k req/min).

### Avoid it when
- The system is private/internal with trusted clients (adds overhead).
- You can scale infinitely without cost concerns (rare).
- You already have sufficient [Backpressure](backpressure.md) and [Load shedding](../operations/load-shedding.md).

## How I would use it (practical)
- **Context:** A SaaS API with free and premium users.
- **Steps:**
  1) Choose algorithm: **token bucket** (allows short bursts) or **sliding window** (strict fairness).
  2) Define limits per tier:
     - Free: 100 requests/minute
     - Premium: 10,000 requests/minute
  3) Implement at [API Gateway](api-gateway.md) or application layer (Redis-based).
  4) Return `429 Too Many Requests` with headers:
     ```
     X-RateLimit-Limit: 100
     X-RateLimit-Remaining: 0
     X-RateLimit-Reset: 1643728800
     Retry-After: 60
     ```
  5) Monitor: rejected requests, burst patterns, user complaints.
  6) Adjust limits based on real usage and SLO budgets.

## Trade-offs / Costs (and their mitigation)
| Trade-off | Mitigation |
|-----------|-----------|
| Legitimate burst traffic throttled | Use token bucket (allows bursts); provide "burst allowance" |
| Per-user tracking overhead | Use efficient data structures (Redis sorted sets, sliding window counters) |
| Coordination in distributed systems | Use centralized rate limiter (Redis) or accept eventual consistency |
| False positives (shared IPs, NAT) | Rate-limit by API key/user ID instead of IP |
| Clients don't handle 429 gracefully | Return clear error messages + `Retry-After` header; document retry policy |

## Failure modes / Edge cases
1. **Thundering herd after rate limit resets:** All clients retry at the same time when window resets.
   - *Mitigation:* Use sliding window or add jitter to client retries.
2. **Distributed rate limiting drift:** Multiple gateway instances have inconsistent counters.
   - *Mitigation:* Centralize state in Redis/Memcached with atomic operations.
3. **Bypass via IP rotation:** Attackers rotate IPs to evade limits.
   - *Mitigation:* Rate-limit by API key or user ID; add CAPTCHA for suspicious patterns.
4. **Rate limiter becomes bottleneck:** Redis/centralized store saturates.
   - *Mitigation:* Use local approximation (per-instance limits) or consistent hashing.
5. **Legitimate users punished during incidents:** High error rates consume quota.
   - *Mitigation:* Only count successful requests; exclude 5xx errors from quota.

## Alternatives
- **[Backpressure](backpressure.md):** Push back on upstream when downstream is saturated.
- **[Load shedding](../operations/load-shedding.md):** Drop requests based on system load, not per-user quotas.
- **[Circuit Breaker](../operations/circuit-breaker.md):** Fails fast when dependency is down (client-side).
- **Queue-based admission control:** Accept all requests into a queue, process at fixed rate.

## Algorithms
### 1. Fixed Window
- Simple counter reset every N seconds.
- **Problem:** Burst at window boundary (100 req at 0:59, 100 at 1:00 = 200 req in 1 sec).

### 2. Sliding Window
- Weighted count of requests in rolling time window.
- **Pros:** Smoother enforcement.
- **Cons:** More complex to implement.

### 3. Token Bucket
- Tokens refill at constant rate; requests consume tokens; allows bursts.
- **Pros:** Flexible, handles bursts gracefully.
- **Cons:** Slightly more state to track.

### 4. Leaky Bucket
- Requests enter queue; processed at fixed rate.
- **Pros:** Smooth output rate.
- **Cons:** Can delay requests (not always desirable for APIs).

**Recommendation:** Use **token bucket** for APIs (allows bursts) or **sliding window** for strict fairness.

## Combinations
Rate Limiting is **almost always used with:**
- **[API Gateway](api-gateway.md):** Centralized enforcement at the edge.
- **[Timeout](../operations/timeouts.md):** Prevents slow requests from holding resources.
- **[Bulkhead](../operations/bulkheads.md):** Isolates impact of one client's traffic.
- **[Observability](../operations/observability-basics.md):** Track 429 rates, top offenders, burst patterns.
- **[Graceful Degradation](../operations/graceful-degradation.md):** Return cached/partial results instead of hard rejection.

**Typical combination:**
- **Public API scenario:** API Gateway + Rate Limiting + Timeout + Retry (client-side with backoff) + Circuit Breaker + Observability

## Prerequisites
- Understanding of [HTTP status codes](../operations/http.md) (429, 503).
- Familiarity with [API design](api-design-basics.md) and [Backpressure](backpressure.md).
- Basic distributed systems knowledge (state sharing, consistency).

## Related topics
- [API Gateway](api-gateway.md): Where rate limiting is typically enforced.
- [Backpressure](backpressure.md): System-level flow control.
- [Load shedding](../operations/load-shedding.md): Drop requests based on system load.
- [Bulkhead](../operations/bulkheads.md): Resource isolation.
- [Observability](../operations/observability-basics.md): Monitor throttling metrics.

## Real-world examples
1. **GitHub API:** 5,000 requests/hour for authenticated users, 60 for unauthenticated.
2. **Stripe:** Rate limits by API key with sliding window; returns `Retry-After` header.
3. **Twitter API:** Tiered limits (free, basic, enterprise) with 15-minute windows.
4. **AWS API Gateway:** Built-in throttling (10,000 req/sec default, configurable).

## Checklist (self-test)
- [ ] I can explain the difference between fixed window, sliding window, and token bucket.
- [ ] I know when to rate-limit by IP vs API key vs user ID.
- [ ] I understand how to handle 429 responses in clients (exponential backoff).
- [ ] I can design rate limits for multi-tier SaaS (free/premium).
- [ ] I can monitor rate limiting effectiveness (rejection rate, false positives).

## Reminders / Key takeaways
- Rate limiting is **preventive**: stops abuse before it overloads the system.
- Always return `Retry-After` header to help clients behave correctly.
- Use token bucket for APIs (allows short bursts), sliding window for strict fairness.
- Combine with [Timeout](../operations/timeouts.md), [Circuit Breaker](../operations/circuit-breaker.md), and [Observability](../operations/observability-basics.md).

## Trap questions (with answers)
### Q1: Should I rate-limit by IP address or API key?
**A:** **Depends on use case**. For public APIs, use **API key** (more precise, avoids NAT/shared IP issues). For unauthenticated endpoints (login, signup), use **IP address** to prevent brute force. For sophisticated setups, combine both: per-user quotas + per-IP caps.

### Q2: What's the difference between rate limiting and backpressure?
**A:** **Rate limiting** is **proactive**: rejects excess requests at the edge based on quotas. **[Backpressure](backpressure.md)** is **reactive**: slows down producers when consumers are saturated. Rate limiting protects from abuse; backpressure protects from overload. Use both: rate limiting at API Gateway, backpressure internally.

### Q3: If I return 429, should the client retry immediately?
**A:** **No**. Clients should implement **exponential backoff** and respect the `Retry-After` header. Immediate retries cause thundering herd and worsen the situation. Recommended: wait `Retry-After` seconds + jitter. See [Retries and backoff](../operations/retries-and-backoff.md).

### Q4: Can rate limiting prevent DDoS attacks?
**A:** **Partially**. Rate limiting mitigates application-layer DDoS (HTTP floods) by capping per-IP/per-user requests. However, it doesn't protect against **network-layer DDoS** (SYN floods, UDP amplification). For full protection, combine with: L4 load balancer, CDN (Cloudflare, AWS Shield), WAF.

### Q5: Should I count failed requests (4xx, 5xx) against the quota?
**A:** **Depends**. For **4xx (client errors)**, yes—prevents attackers from probing without penalty. For **5xx (server errors)**, **no**—don't punish users for your failures. Exception: 429 itself should not consume quota (allows graceful retry).
