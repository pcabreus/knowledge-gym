---
id: api-composition
title: "API Composition"
type: pattern
status: learning
importance: 65
difficulty: medium
tags: [architecture, api-design, microservices, aggregation]
canonical: true
aliases: [api aggregation, data aggregation]
created_at: 2026-01-26
updated_at: 2026-01-26
---

# API Composition

## TL;DR (BLUF)
- Aggregates data from multiple microservices into a single response.
- Use it to avoid "chatty clients" and reduce round-trips.
- Trade-off: added latency and complexity; risk of coupling services.

## Definition
**What it is:** A pattern where a service (typically a [BFF](backend-for-frontend.md) or API layer) calls multiple downstream services in parallel or sequentially and combines their responses into a single payload for the client.

**Key terms:** aggregation, composition, data assembly, parallel calls, partial responses, chatty client.

## Why it matters
- Reduces client complexity: one request instead of N.
- Decreases latency for clients (especially mobile on slow networks).
- Improves user experience for "composed" screens (order details, dashboards, home feeds).
- Centralizes error handling and retries.

## Scope & Non-goals
**In scope:** aggregating data from multiple services, parallel calls, error handling for partial responses.

**Out of scope / NOT solved by this:**
- Distributed transactions (use [Saga](saga-pattern.md))
- Strong consistency across services (eventual consistency only)
- Business logic (aggregation is data assembly, not domain logic)

## Mental model / Intuition
- Like assembling a meal from different food stalls at a food court: you coordinate orders from multiple vendors and deliver one tray.
- In APIs: `/order/123` needs data from order-service, user-service, payment-service, and inventory-service.

## Decision rules (When to use / When not to use)
### Use it when
- A client screen needs data from 3+ services (order details, user profile, recommendations).
- Mobile or web clients are "chatty" (many round-trips).
- You control all downstream services (same organization).
- Aggregation logic is simple (no complex joins or transformations).

### Avoid it when
- Downstream services are owned by external teams and may change independently.
- Aggregation requires complex business logic (use a dedicated service instead).
- One slow service blocks the entire response (consider [Graceful Degradation](../operations/graceful-degradation.md)).
- Clients can efficiently fetch data themselves (GraphQL, local caching).

## How I would use it (practical)
- **Context:** E-commerce "Order Details" page needs: order info, user profile, payment status, shipping tracking.
- **Steps:**
  1) Implement in [BFF](backend-for-frontend.md) (web-bff or mobile-bff).
  2) Call services **in parallel** using async I/O (Go: goroutines; Node: Promises; Java: CompletableFuture).
     ```go
     var wg sync.WaitGroup
     var order Order
     var user User
     var payment Payment
     wg.Add(3)
     go func() { order = fetchOrder(ctx, orderID); wg.Done() }()
     go func() { user = fetchUser(ctx, userID); wg.Done() }()
     go func() { payment = fetchPayment(ctx, paymentID); wg.Done() }()
     wg.Wait()
     ```
  3) Set strict [Timeouts](../operations/timeouts.md) (e.g., 500ms per service).
  4) Implement [Graceful Degradation](../operations/graceful-degradation.md): if payment-service fails, return order + user (omit payment).
  5) Add [Circuit Breaker](../operations/circuit-breaker.md) per downstream service.
  6) Monitor: aggregation latency p95/p99, partial response rate, downstream error rates.

## Trade-offs / Costs (and their mitigation)
| Trade-off | Mitigation |
|-----------|-----------|
| Latency = slowest service (if sequential) | Use **parallel calls**; set aggressive timeouts |
| One service failure breaks entire response | Use [Graceful Degradation](../operations/graceful-degradation.md) (partial responses) |
| Coupling to multiple service contracts | Version APIs carefully; use [API versioning](../system-design/versioning-apis-and-events.md) |
| Aggregator becomes bottleneck | Scale horizontally; cache expensive aggregations |
| N+1 query problem (loops calling services) | Use batching or data loaders (GraphQL-style) |

## Failure modes / Edge cases
1. **Cascading timeout:** Aggregator timeout too short; all downstream calls fail.
   - *Mitigation:* Set aggregator timeout > max(downstream timeouts) + buffer.
2. **Partial response inconsistency:** User sees order but not payment status.
   - *Mitigation:* Mark partial responses in payload; show "loading..." or fallback.
3. **Thundering herd on cache miss:** All aggregations trigger simultaneous downstream calls.
   - *Mitigation:* Use request coalescing or short-term in-memory cache.
4. **Service A depends on data from Service B:** Sequential calls increase latency.
   - *Mitigation:* Redesign to reduce dependencies; use event-driven architecture.
5. **Aggregator becomes domain service:** Teams add business logic instead of pure aggregation.
   - *Mitigation:* Enforce policy: aggregation is **data assembly only**.

## Alternatives
- **GraphQL:** Clients specify exactly what fields they need; server resolves dependencies.
- **Client-side composition:** Client calls services directly (simple but chattier).
- **[CQRS](cqrs.md) + Read Model:** Pre-aggregate data in a read-optimized store.
- **Event-driven materialized views:** Downstream services publish events; aggregator maintains local view.

## Combinations
API Composition is **almost always used with:**
- **[BFF (Backend for Frontend)](backend-for-frontend.md):** BFFs are the natural place for composition.
- **[Timeout](../operations/timeouts.md):** Prevent slow services from blocking aggregation.
- **[Circuit Breaker](../operations/circuit-breaker.md):** Fail fast when downstream is unhealthy.
- **[Graceful Degradation](../operations/graceful-degradation.md):** Return partial data if non-critical services fail.
- **[Observability](../operations/observability-basics.md):** Track downstream latency, error rates, partial response rates.

**Typical combination:**
- **Composed UI scenario:** BFF + API Composition + Timeout + Circuit Breaker + Graceful Degradation + Observability

## Prerequisites
- Understanding of [Microservices design](microservices-design-basics.md).
- Familiarity with async I/O and parallel execution.
- Knowledge of [Timeouts](../operations/timeouts.md) and [Circuit Breaker](../operations/circuit-breaker.md).

## Related topics
- [BFF (Backend for Frontend)](backend-for-frontend.md): Where composition typically happens.
- [Graceful Degradation](../operations/graceful-degradation.md): Handle partial failures.
- [CQRS](cqrs.md): Pre-aggregate data in read models.
- [GraphQL](../system-design/api-design-basics.md): Alternative to explicit composition.
- [Timeout](../operations/timeouts.md): Critical for aggregation.

## Real-world examples
1. **Netflix:** BFFs aggregate user profile + viewing history + recommendations.
2. **Amazon Product Page:** Aggregates product details + reviews + inventory + recommendations.
3. **Uber Trip Details:** Aggregates trip info + driver profile + payment + route.

## Checklist (self-test)
- [ ] I can identify when to use API composition vs client-side fetching.
- [ ] I know how to implement parallel calls to reduce latency.
- [ ] I can design partial response handling (graceful degradation).
- [ ] I understand the N+1 query problem and how to avoid it.
- [ ] I can monitor aggregation health (latency, error rates, partial responses).

## Reminders / Key takeaways
- Always use **parallel calls** to minimize latency.
- Set strict [Timeouts](../operations/timeouts.md) per downstream service.
- Use [Graceful Degradation](../operations/graceful-degradation.md): partial responses are better than total failure.
- API Composition is **data assembly**, not business logic.

## Trap questions (with answers)
### Q1: Should I use API composition or GraphQL?
**A:** **Depends on control and complexity**. Use **API composition** when you control all services and aggregation is simple (fixed queries). Use **GraphQL** when clients need flexible queries or when aggregation logic is complex. GraphQL adds overhead (query parsing, resolver logic) but gives clients more power. For internal BFFs, composition is simpler.

### Q2: What if one service is slow—should I wait for it?
**A:** **No, use timeouts and graceful degradation**. Set a strict [Timeout](../operations/timeouts.md) (e.g., 500ms). If a non-critical service times out, return partial data with a flag (`"payment": null, "paymentUnavailable": true`). Critical services (e.g., order details) failing should return 503. See [Graceful Degradation](../operations/graceful-degradation.md).

### Q3: How do I avoid making N+1 queries in aggregation?
**A:** **Use batching or data loaders**. Instead of:
```
for user in users:
    fetch orders(user.id)  // N calls
```
Do:
```
userIds = [u.id for u in users]
orders = fetchOrdersBatch(userIds)  // 1 call
```
Or use GraphQL-style data loaders that batch and deduplicate requests within a single request context.

### Q4: Can API composition work for write operations (POST, PUT)?
**A:** **Generally no**. Composition is for reads. For writes across multiple services, use [Saga pattern](saga-pattern.md) (orchestrated or choreographed transactions with compensations). Aggregation assumes **read-only** and **eventual consistency**.

### Q5: Should I cache aggregated responses?
**A:** **Yes, if data changes infrequently**. Cache at the aggregation layer (Redis, in-memory) with appropriate TTLs. Be careful with user-specific data (privacy) and ensure cache invalidation works. For frequently changing data (stock prices, live scores), use short TTLs or skip caching. See [Caching strategy](../system-design/caching-strategy.md).
