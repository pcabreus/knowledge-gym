---
id: backend-for-frontend
title: "Backend for Frontend (BFF)"
type: pattern
status: learning
importance: 70
difficulty: medium
tags: [architecture, api-design, microservices]
canonical: true
aliases: [bff]
created_at: 2026-01-26
updated_at: 2026-01-26
---

# Backend for Frontend (BFF)

## TL;DR (BLUF)
- A specialized backend service tailored to the needs of a specific client (web, mobile, partners).
- Use it when different clients need different payloads, caching, or API contracts.
- Trade-off: code duplication across BFFs and increased operational overhead.

## Definition
**What it is:** A pattern where each type of frontend (web, mobile, desktop, partners) has its own dedicated backend service that aggregates, transforms, and optimizes data from downstream microservices.

**Key terms:** backend for frontend, client-specific API, aggregation layer, payload optimization, independent evolution.

## Why it matters
- Decouples frontend experience from backend service structure.
- Allows each client to evolve independently (web releases vs mobile releases).
- Optimizes payloads per client (mobile needs less data, web needs more).
- Prevents "lowest common denominator" API design.

## Scope & Non-goals
**In scope:** client-specific aggregation, transformation, caching, authentication delegation, payload optimization.

**Out of scope / NOT solved by this:**
- Cross-cutting concerns (auth, rate limiting) → [API Gateway](../system-design/api-gateway.md)
- Business logic (belongs in domain services)
- Service mesh concerns → [Sidecar](../operations/sidecar.md)

## Mental model / Intuition
- Think of a restaurant with different menus for dine-in, takeout, and delivery. Same kitchen (services), different presentation.
- Web-BFF: full details, rich filtering.
- Mobile-BFF: minimal payload, optimized for slow networks.
- Partner-BFF: specific fields, strict versioning.

## Decision rules (When to use / When not to use)
### Use it when
- Different clients (web, mobile, IoT) have different payload/performance needs.
- Clients release independently (web every week, mobile every month).
- You need client-specific caching, rate limiting, or security policies.
- Aggregating from multiple services creates "chatty client" problems.
- You want to experiment with new features for one client without affecting others.

### Avoid it when
- You have a single client or clients share identical needs.
- Aggregation logic is simple and can live in the client.
- Operational overhead of multiple BFFs outweighs benefits.
- Teams lack resources to maintain separate codebases.

## How I would use it (practical)
- **Context:** E-commerce platform with web and mobile clients, calling user, product, order, and recommendation services.
- **Steps:**
  1) Create two BFFs: `web-bff` and `mobile-bff`.
  2) **Web-BFF:**
     - Aggregates user profile + recent orders + recommendations.
     - Returns full product details (images, descriptions, reviews).
     - Implements server-side pagination.
  3) **Mobile-BFF:**
     - Returns minimal product details (thumbnail, title, price).
     - Implements aggressive caching (CDN + Redis).
     - Uses optimized image formats (WebP).
  4) Both BFFs call the same downstream services but transform responses.
  5) [API Gateway](../system-design/api-gateway.md) routes: `/web/*` → web-bff, `/mobile/*` → mobile-bff.
  6) Monitor: latency, cache hit rate, error rate per BFF.

## Trade-offs / Costs (and their mitigation)
| Trade-off | Mitigation |
|-----------|-----------|
| Code duplication (shared aggregation logic) | Extract common libraries/SDKs; use shared data access layer |
| Operational overhead (deploy, monitor N BFFs) | Use templated deployment pipelines; shared observability stack |
| Risk of divergence (BFFs drift apart) | Enforce shared contracts with downstream services; automated integration tests |
| Increased latency (extra hop) | Optimize with caching, parallel calls, HTTP/2 |
| Harder to enforce consistency | Use shared domain services; BFFs only do transformation |

## Failure modes / Edge cases
1. **BFF becomes a mini-monolith:** Teams add business logic instead of transformation.
   - *Mitigation:* Enforce policy: BFFs are thin aggregation layers, not domain services.
2. **Cascading failures from downstream:** If one service is slow, BFF blocks.
   - *Mitigation:* Combine with [Timeout](../operations/timeouts.md), [Circuit Breaker](../operations/circuit-breaker.md), and [Graceful Degradation](../operations/graceful-degradation.md).
3. **Stale cache inconsistencies:** Mobile BFF caches aggressively, shows old data.
   - *Mitigation:* Use cache invalidation (events, TTLs); monitor staleness metrics.
4. **Ownership confusion:** Who owns the BFF? Frontend team or backend team?
   - *Mitigation:* Clear ownership model (typically frontend team owns BFF).
5. **N+1 query problem:** BFF calls downstream service in a loop.
   - *Mitigation:* Use batching, parallel calls, or GraphQL-style data loaders.

## Alternatives
- **API Gateway with transformation:** Lighter but limited to simple transformations.
- **GraphQL:** Clients query exactly what they need; reduces BFF logic.
- **API Composition in client:** Client calls multiple services directly (chattier, slower).
- **Shared backend API:** One API for all clients (forces lowest common denominator).

## Combinations
BFF is **almost always used with:**
- **[API Gateway](../system-design/api-gateway.md):** Routes traffic to appropriate BFF.
- **[API Composition](api-composition.md):** BFF aggregates data from multiple services.
- **[Timeout](../operations/timeouts.md):** Prevents slow downstream calls from blocking.
- **[Circuit Breaker](../operations/circuit-breaker.md):** Fails fast when downstream is unhealthy.
- **[Graceful Degradation](../operations/graceful-degradation.md):** Returns partial data if non-critical services fail.
- **[Observability](../operations/observability-basics.md):** Track latency, error rate per BFF.

**Typical combination:**
- **Mobile + Web scenario:** API Gateway + BFF (web/mobile) + API Composition (in BFF) + Observability

## Prerequisites
- Understanding of [Microservices design](microservices-design-basics.md).
- Familiarity with [API design](../system-design/api-design-basics.md) and [REST/GraphQL](../system-design/api-design-basics.md).
- Knowledge of [Caching](../system-design/caching-fundamentals.md) and [HTTP](../operations/http.md).

## Related topics
- [API Gateway](../system-design/api-gateway.md): Routes to BFFs.
- [API Composition](api-composition.md): Aggregation pattern used within BFFs.
- [Graceful Degradation](../operations/graceful-degradation.md): Partial responses when services fail.
- [Microservices design](microservices-design-basics.md): Context for BFF pattern.
- [Caching strategy](../system-design/caching-strategy.md): Optimize BFF performance.

## Real-world examples
1. **Netflix:** Separate BFFs for web, mobile, TV clients—each optimized for device constraints.
2. **SoundCloud:** Web-BFF aggregates playlists + recommendations; mobile-BFF returns minimal data.
3. **Spotify:** Mobile-BFF uses aggressive caching and optimized image sizes.

## Checklist (self-test)
- [ ] I can explain when BFF is needed vs when API Gateway is enough.
- [ ] I understand the ownership model (frontend team owns BFF).
- [ ] I can design aggregation logic without putting business rules in BFF.
- [ ] I know how to handle partial failures (circuit breaker, graceful degradation).
- [ ] I can monitor BFF health (latency, cache hit rate, error rate).

## Reminders / Key takeaways
- BFF is a **thin aggregation layer**, not a domain service.
- One BFF per client type: web-BFF, mobile-BFF, partner-BFF.
- Always combine with [Timeout](../operations/timeouts.md), [Circuit Breaker](../operations/circuit-breaker.md), and [Graceful Degradation](../operations/graceful-degradation.md).
- Use [API Gateway](../system-design/api-gateway.md) to route traffic to BFFs.

## Trap questions (with answers)
### Q1: Can I put business logic in a BFF?
**A:** **No**. BFFs should only **aggregate, transform, and optimize** data from downstream services. Business logic (validation, calculations, domain rules) belongs in domain services. Violating this creates tight coupling and duplicates logic across BFFs.

### Q2: What's the difference between BFF and API Gateway?
**A:** **API Gateway** is infrastructure: routing, auth, rate limiting, TLS termination for **all clients**. **BFF** is a service: client-specific aggregation and transformation. You use both: Gateway routes to BFFs (`/web/*` → web-bff, `/mobile/*` → mobile-bff). See [API Gateway](../system-design/api-gateway.md).

### Q3: Should I create a BFF for every single page/screen?
**A:** **No**. Create BFFs per **client type** (web, mobile, partners), not per page. Within a BFF, different endpoints handle different screens. Creating too many BFFs increases operational overhead without significant benefit.

### Q4: Can BFFs share code?
**A:** **Yes, but carefully**. Extract shared logic into libraries (HTTP clients, auth helpers, data mappers). However, don't force shared aggregation logic—BFFs exist precisely to diverge based on client needs. Balance: shared infrastructure, independent business transformations.

### Q5: How do I prevent BFFs from becoming bottlenecks?
**A:** 
1. Use **parallel calls** to downstream services (not sequential).
2. Implement **caching** (Redis, CDN) for expensive aggregations.
3. Add [Circuit Breaker](../operations/circuit-breaker.md) and [Timeout](../operations/timeouts.md) to fail fast.
4. Scale BFFs horizontally (stateless design).
5. Monitor latency and optimize slow endpoints. See [Performance basics](../system-design/performance-basics.md).
