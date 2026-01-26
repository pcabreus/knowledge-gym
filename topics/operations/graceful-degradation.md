---
id: graceful-degradation
title: "Graceful Degradation"
type: pattern
status: learning
importance: 75
difficulty: medium
tags: [operations, reliability, resilience, user-experience]
canonical: true
aliases: [partial failure handling, fallback pattern]
created_at: 2026-01-26
updated_at: 2026-01-26
---

# Graceful Degradation

## TL;DR (BLUF)
- Return partial or cached responses when non-critical services fail instead of complete failure.
- Use it to improve availability and user experience during partial outages.
- Trade-off: users get incomplete data; adds complexity to decide what's "critical."

## Definition
**What it is:** A resilience pattern where a system continues to function in a limited capacity when dependencies fail, by returning cached data, omitting non-critical features, or using fallback values.

**Key terms:** partial response, fallback, degraded mode, stale cache, feature toggles, eventual consistency, availability over completeness.

## Why it matters
- Improves perceived availability: "something works" is better than "nothing works."
- Reduces blast radius of downstream failures.
- Enhances user experience during incidents (show order history even if recommendations fail).
- Complements [Circuit Breaker](circuit-breaker.md) and [Timeout](timeouts.md).

## Scope & Non-goals
**In scope:** partial responses, fallback logic, cached data, feature flags, static defaults.

**Out of scope / NOT solved by this:**
- Preventing failures (use [Circuit Breaker](circuit-breaker.md), [Retry](retries-and-backoff.md))
- Strong consistency (eventual consistency only)
- Business transactions (use [Saga](../architecture/saga-pattern.md))

## Mental model / Intuition
- Like a restaurant: if out of dessert, still serve main course (don't close the restaurant).
- In APIs: if recommendations service is down, show product page without recommendations.

## Decision rules (When to use / When not to use)
### Use it when
- Failures in non-critical features shouldn't block critical paths (checkout can work without recommendations).
- Cached/stale data is acceptable for some use cases (news feed, product catalogs).
- User experience benefits from partial functionality (show 80% of data vs error page).
- Third-party integrations are unreliable.

### Avoid it when
- All dependencies are critical (payment processing, auth).
- Stale data is dangerous (financial data, real-time pricing).
- Users expect complete, consistent responses (banking, healthcare).
- Complexity of fallback logic outweighs benefits.

## How I would use it (practical)
- **Context:** E-commerce product page needs: product details (critical), reviews (important), recommendations (nice-to-have).
- **Steps:**
  1) Classify dependencies:
     - **Critical:** product-service (without it, no page).
     - **Important:** review-service (degrade: show "Reviews unavailable").
     - **Optional:** recommendation-service (omit silently).
  2) Implement with [Timeout](timeouts.md) + fallback:
     ```go
     product := fetchProduct(ctx, productID) // Critical: fail if this fails
     
     reviews, err := fetchReviews(ctx, productID)
     if err != nil {
         reviews = cachedReviews(productID) // Fallback: use cache
     }
     
     recommendations, err := fetchRecommendations(ctx, productID)
     if err != nil {
         recommendations = nil // Degrade: omit recommendations
     }
     
     return ProductPage{
         Product: product,
         Reviews: reviews,
         Recommendations: recommendations,
         Degraded: recommendations == nil, // Signal to UI
     }
     ```
  3) UI handles `Degraded` flag: hide recommendations section or show "Recommendations temporarily unavailable."
  4) Cache aggressively for non-critical data (TTL: 1-24 hours).
  5) Monitor: degradation frequency, cache hit rate, user impact.

## Trade-offs / Costs (and their mitigation)
| Trade-off | Mitigation |
|-----------|-----------|
| Users get incomplete data | Clearly communicate degradation in UI ("Some features unavailable") |
| Stale cache can mislead users | Set appropriate TTLs; show "last updated" timestamp |
| Complexity deciding what's critical | Document criticality tiers; use feature flags for experimentation |
| Silent failures (logs ignored) | Alert on degradation rate; track SLO impact |
| Cached data privacy concerns | Respect user permissions; encrypt cached data |

## Failure modes / Edge cases
1. **Over-degradation:** Too many features degraded; user experience unacceptable.
   - *Mitigation:* Define minimum viable response; fail fast if too degraded.
2. **Stale cache poisoning:** Cached bad data served for hours.
   - *Mitigation:* Validate cached data before serving; short TTLs for critical data.
3. **Cascade of degradations:** Multiple services degrade simultaneously.
   - *Mitigation:* Monitor overall degradation; alert when multiple services affected.
4. **Users unaware of degradation:** Silent omissions confuse users.
   - *Mitigation:* Show banners ("Some features limited due to maintenance").
5. **Feature flag drift:** Degradation mode stays enabled after incident.
   - *Mitigation:* Automate feature flag expiry; post-incident review.

## Alternatives
- **Fail fast (503):** Return error immediately; simpler but worse UX.
- **[Circuit Breaker](circuit-breaker.md):** Prevents cascading failures; complements degradation.
- **[Retry](retries-and-backoff.md):** Attempt recovery; use before degradation.
- **Queue-based processing:** Accept request, process later (async).

## Implementation patterns
### 1. Static fallback
- Return hardcoded default: empty list, placeholder image, default config.
- **Use:** non-critical features (recommendations, ads).

### 2. Cached response
- Serve stale data from cache (Redis, CDN).
- **Use:** content that changes infrequently (product catalogs, news).

### 3. Partial response
- Return subset of data (omit failed sections).
- **Use:** composite pages (dashboard, aggregated views).

### 4. Feature flag
- Disable feature entirely during incidents.
- **Use:** experimental features, non-essential functionality.

### 5. User notification
- Show banner: "Search temporarily unavailable."
- **Use:** when degradation significantly impacts UX.

## Combinations
Graceful Degradation is **almost always used with:**
- **[Timeout](timeouts.md):** Detect failure quickly.
- **[Circuit Breaker](circuit-breaker.md):** Fail fast when dependency is unhealthy.
- **[Caching](../system-design/caching-fundamentals.md):** Serve stale data as fallback.
- **[Observability](observability-basics.md):** Track degradation rate, impact on SLO.
- **[API Composition](../architecture/api-composition.md):** Handle partial failures in aggregation.

**Typical combination:**
- **Resilient service scenario:** Timeout + Circuit Breaker + Graceful Degradation + Caching + Observability

## Prerequisites
- Understanding of [Caching fundamentals](../system-design/caching-fundamentals.md).
- Familiarity with [Circuit Breaker](circuit-breaker.md) and [Timeout](timeouts.md).
- Knowledge of [Observability](observability-basics.md) (monitoring degradation).

## Related topics
- [Circuit Breaker](circuit-breaker.md): Fails fast when dependency is down.
- [Timeout](timeouts.md): Detects slow dependencies.
- [Caching strategy](../system-design/caching-strategy.md): Fallback data source.
- [API Composition](../architecture/api-composition.md): Aggregate with partial failures.
- [Observability](observability-basics.md): Monitor degradation metrics.

## Real-world examples
1. **Netflix:** If personalization service fails, show generic recommendations.
2. **Amazon:** Product page loads without reviews if review service is down.
3. **Twitter:** Timeline loads even if trending topics fail.
4. **Spotify:** App works offline with cached playlists (extreme degradation).

## Checklist (self-test)
- [ ] I can classify dependencies as critical/important/optional.
- [ ] I know how to implement fallback logic (cache, static defaults, omission).
- [ ] I understand when to show partial data vs fail completely.
- [ ] I can monitor degradation frequency and user impact.
- [ ] I can communicate degradation clearly to users.

## Reminders / Key takeaways
- **Availability > Completeness** for non-critical features.
- Always classify dependencies: critical, important, optional.
- Cache aggressively for non-critical data.
- Clearly communicate degradation to users (UI banners, response flags).
- Monitor degradation rate and impact on [SLO](service-level-objective.md).

## Trap questions (with answers)
### Q1: Should I always degrade instead of returning an error?
**A:** **No**. Only degrade **non-critical features**. For critical paths (auth, payment, order creation), **fail fast with clear error**. Degradation is for improving UX when partial functionality is acceptable. Don't degrade safety-critical systems (healthcare, aviation).

### Q2: How long can I serve stale cached data?
**A:** **Depends on use case**. For product catalogs, 1-24 hours is fine. For stock prices, 1 minute may be too long. Set TTL based on **data sensitivity** and **user expectations**. Always show "last updated" timestamp for stale data.

### Q3: What's the difference between graceful degradation and circuit breaker?
**A:** **[Circuit Breaker](circuit-breaker.md)** **detects failure** and fails fast to prevent cascades. **Graceful Degradation** **handles failure** by returning partial/cached data. Use both: circuit breaker opens → graceful degradation returns fallback. Circuit breaker is a **failure detector**; degradation is a **failure handler**.

### Q4: Should I alert on every degradation?
**A:** **No, but track metrics**. Alert when:
1. Degradation rate exceeds threshold (e.g., >10% of requests).
2. Critical services degrade (not just optional features).
3. Multiple services degrade simultaneously.
Track all degradations in dashboards for post-incident analysis. See [Observability](observability-basics.md).

### Q5: Can I use graceful degradation for write operations (POST, PUT)?
**A:** **Rarely**. Degradation is primarily for **reads**. For writes, you usually can't "partially" create an order or "cache" a payment. Instead:
- Queue write for later processing (async).
- Return 503 with `Retry-After` header.
- Use [Saga pattern](../architecture/saga-pattern.md) for distributed transactions.
Exception: Non-critical writes (analytics, logs) can be dropped during incidents.
