---
id: microservices-pattern-combinations
title: "Microservices Pattern Combinations"
type: reference
status: learning
importance: 90
difficulty: hard
tags: [architecture, microservices, patterns, system-design, operations]
canonical: true
aliases: [pattern combinations, canonical combinations]
created_at: 2026-01-26
updated_at: 2026-01-26
---

# Microservices Pattern Combinations

## Overview
This document maps **real-world scenarios** to **proven pattern combinations**. Patterns rarely work in isolation—they combine to solve complex distributed systems problems.

## Core Pattern Groups

### "Santísima Trinidad" (Resilience Trinity)
For **every** synchronous call to external/downstream services:
- **[Timeout](../operations/timeouts.md):** Detect slow calls quickly
- **[Retry](../operations/retries-and-backoff.md):** Handle transient failures with exponential backoff + jitter
- **[Circuit Breaker](../operations/circuit-breaker.md):** Fail fast when downstream is unhealthy

**Add to this:**
- **[Bulkhead](../operations/bulkheads.md):** Isolate resources per dependency
- **[Observability](../operations/observability-basics.md):** Track latency, error rates, circuit state

**Why:** Prevents cascading failures, saturated connection pools, and uncontrolled retries.

---

## Pattern Combinations by Scenario

### A. Edge / API & Client Layer

#### Public API (Web/Partners)
**Goal:** Security, traffic control, client simplicity

**Patterns:**
1. **[API Gateway](../system-design/api-gateway.md):** Single entry point
2. **[Rate Limiting](../system-design/rate-limiting.md):** Protect from abuse
3. **[Timeout](../operations/timeouts.md):** Prevent hanging requests
4. **[Retry](../operations/retries-and-backoff.md):** Handle transient failures (with backoff)
5. **[Circuit Breaker](../operations/circuit-breaker.md):** Fail fast when backends down
6. **[Observability](../operations/observability-basics.md):** Monitor edge metrics

**Real example:** Stripe API
- API Gateway handles auth, routing, versioning
- Rate limiting: 100 req/sec per key
- Circuit breaker opens if payment processor down
- Observability: track p95 latency, error rate by endpoint

---

#### Mobile + Web with Different Needs
**Goal:** Distinct payloads, independent releases

**Patterns:**
1. **[BFF](../architecture/backend-for-frontend.md):** One per client (web-BFF, mobile-BFF)
2. **[API Composition](../architecture/api-composition.md):** Aggregate data within BFF
3. **[Observability](../operations/observability-basics.md):** Monitor per-BFF metrics

**Why BFF over shared API:**
- Mobile needs minimal payloads (slow networks)
- Web needs rich data (filters, pagination)
- Different release cadences (web weekly, mobile monthly)

**Real example:** Netflix
- Web-BFF: rich metadata, high-res images
- Mobile-BFF: compressed payloads, aggressive caching
- TV-BFF: optimized for remote control navigation

---

#### Composed UI Screens (Order Details, Home Feed)
**Goal:** Avoid "chatty client" (many round-trips)

**Patterns:**
1. **[API Composition](../architecture/api-composition.md):** Aggregate in BFF or gateway
2. **[Timeout](../operations/timeouts.md):** Cap aggregation time
3. **[Circuit Breaker](../operations/circuit-breaker.md):** Isolate failures
4. **[Graceful Degradation](../operations/graceful-degradation.md):** Return partial data if non-critical services fail

**Implementation:**
```
Order Details = order-service + user-service + payment-service + shipping-service
- Call in parallel (reduce latency)
- If shipping-service fails → show order without tracking (degraded)
- If order-service fails → return 503 (critical)
```

**Real example:** Amazon product page
- Aggregates: product, reviews, recommendations, inventory
- Recommendations fail → page still loads (degraded)
- Product details fail → error page (critical)

---

#### API Evolution Without Breaking Clients
**Goal:** Compatibility, gradual migration

**Patterns:**
1. **[API Gateway](../system-design/api-gateway.md):** Route by version
2. **[BFF](../architecture/backend-for-frontend.md):** Transform per client needs
3. **[API Versioning](../system-design/versioning-apis-and-events.md):** URL or header versioning
4. **[Observability](../operations/observability-basics.md):** Track usage of old vs new versions

**Strategy:**
- Gateway routes `/v1/*` → legacy, `/v2/*` → new
- Monitor v1 usage; deprecate when < 5% traffic
- Use BFF to adapt v1 clients to v2 backend

---

### B. Resilience / Partial Failures

#### Downstream Slow or Intermittent
**Goal:** Prevent cascading failures

**Patterns:**
1. **[Timeout](../operations/timeouts.md):** Fail fast
2. **[Retry](../operations/retries-and-backoff.md):** Exponential backoff + jitter
3. **[Circuit Breaker](../operations/circuit-breaker.md):** Stop calling when broken
4. **[Bulkhead](../operations/bulkheads.md):** Isolate connection pools
5. **[Observability](../operations/observability-basics.md):** Track timeout rate, circuit state

**Example:** Contact platform calling voice API
- Timeout: 2s
- Retry: 3 attempts with exponential backoff
- Circuit breaker: open after 5 consecutive failures
- Bulkhead: separate pool for voice API (10 connections)
- Fallback: queue call for later retry (async)

---

#### Traffic Spikes / Load Protection
**Goal:** Protect limited resources

**Patterns:**
1. **[Rate Limiting](../system-design/rate-limiting.md):** Cap per user/IP
2. **[Bulkhead](../operations/bulkheads.md):** Partition resources
3. **[Timeout](../operations/timeouts.md):** Drop slow requests
4. **[Graceful Degradation](../operations/graceful-degradation.md):** Disable non-critical features

**Example:** Black Friday sale
- Rate limit: 1000 req/min per user
- Bulkhead: separate pools for checkout vs browsing
- Graceful degradation: disable recommendations, show cached inventory

---

#### Third-Party Integration (External APIs)
**Goal:** Isolate risk from unreliable dependencies

**Patterns:**
1. **[Circuit Breaker](../operations/circuit-breaker.md):** Fail fast
2. **[Timeout](../operations/timeouts.md):** Strict limits
3. **[Retry](../operations/retries-and-backoff.md):** Limited retries
4. **[Graceful Degradation](../operations/graceful-degradation.md):** Fallback to cache or default
5. **[Observability](../operations/observability-basics.md):** Monitor third-party health
6. **[Anti-Corruption Layer](../architecture/anti-corruption-layer.md):** Protect domain from external models

**Example:** Payment gateway (Stripe)
- ACL translates Stripe response to domain `Payment`
- Circuit breaker opens if Stripe down
- Fallback: queue payment for retry
- Observability: alert if Stripe error rate > 5%

---

#### Service Mesh (mTLS, Retries, Tracing)
**Goal:** Standardize cross-cutting concerns

**Patterns:**
1. **[Sidecar](../operations/sidecar.md):** Envoy proxy per pod
2. **[Observability](../operations/observability-basics.md):** Distributed tracing
3. **[Timeout](../operations/timeouts.md):** Configured in sidecar
4. **[Retry](../operations/retries-and-backoff.md):** Configured in sidecar
5. **[Circuit Breaker](../operations/circuit-breaker.md):** Configured in sidecar

**Implementation:** Istio or Linkerd
- No code change in services
- Policies defined in YAML
- mTLS automatic between services
- Tracing auto-injected (Jaeger/Zipkin)

---

### C. Data, Consistency, and Async (Event-Driven)

#### Multi-Service Transaction (Checkout)
**Goal:** Eventual consistency with compensation

**Patterns:**
1. **[Saga](../architecture/saga-pattern.md):** Orchestrate steps + compensations
2. **[Outbox](../architecture/outbox-pattern.md):** Reliable event publishing
3. **[Idempotency](../operations/idempotency.md):** Safe retries
4. **[Retry](../operations/retries-and-backoff.md):** Handle transient failures
5. **[Observability](../operations/observability-basics.md):** Track saga state

**Example:** E-commerce checkout
```
1. Reserve inventory (inventory-service)
2. Charge payment (payment-service)
3. Create order (order-service)

If payment fails:
- Compensate: release inventory reservation
```

**Outbox ensures:**
- Events published atomically with DB writes
- No lost events (dual-write problem solved)

---

#### High Read Load, Complex Writes
**Goal:** Scale reads independently

**Patterns:**
1. **[CQRS](../architecture/cqrs.md):** Separate read/write models
2. **[Database per Service](../architecture/database-per-service.md):** Isolated data
3. **[Outbox](../architecture/outbox-pattern.md):** Sync write → read models
4. **[Observability](../operations/observability-basics.md):** Monitor sync lag

**Example:** News platform
- Write model: article-service (Postgres)
- Read model: search-service (Elasticsearch)
- Write publishes `ArticlePublished` event
- Search-service subscribes, updates index

---

#### Full Audit Trail / Replay (Event Sourcing)
**Goal:** Complete history, time travel

**Patterns:**
1. **[Event Sourcing](../architecture/event-sourcing.md):** Store events, not state
2. **[CQRS](../architecture/cqrs.md):** Build read models from events
3. **[Observability](../operations/observability-basics.md):** Monitor event replay

**Example:** Banking system
- Store: `AccountCreated`, `MoneyDeposited`, `MoneyWithdrawn`
- Current balance = replay all events
- Audit: full transaction history immutable
- Time travel: "What was balance on Jan 1?"

---

#### Reliable Event Integration
**Goal:** Never lose events, handle duplicates

**Patterns:**
1. **[Outbox](../architecture/outbox-pattern.md):** Atomic event publishing
2. **[Retry](../operations/retries-and-backoff.md):** Retry failed consumers
3. **[Idempotency](../operations/idempotency.md):** Deduplicate events
4. **[Data Reconciliation](../operations/data-reconciliation.md):** Detect and fix drift
5. **[Observability](../operations/observability-basics.md):** Monitor event lag, DLQ depth

**Implementation:**
- Producer writes to DB + outbox table in same transaction
- Relay process publishes from outbox → Kafka
- Consumer uses idempotency key to dedupe
- Dead-letter queue for poison messages
- Periodic reconciliation catches edge cases (lost events, bugs)

---

### D. Architecture Evolution / Legacy

#### Migrate Monolith to Microservices
**Goal:** Incremental, low-risk migration

**Patterns:**
1. **[Strangler Fig](../architecture/strangler-fig.md):** Incremental replacement
2. **[API Gateway](../system-design/api-gateway.md):** Route to new vs legacy
3. **[Anti-Corruption Layer](../architecture/anti-corruption-layer.md):** Protect new services
4. **[Observability](../operations/observability-basics.md):** Monitor migration progress

**Phases:**
```
Phase 1: Gateway in front of monolith
Phase 2: Extract product-service → route /products/* to new service
Phase 3: Extract order-service → route /orders/* to new service
Phase 4: Repeat until monolith empty
Phase 5: Decommission monolith
```

**ACL protects:** New services from legacy's poor data model

---

#### Integrate with Legacy or Third-Party
**Goal:** Protect domain from external mess

**Patterns:**
1. **[Anti-Corruption Layer](../architecture/anti-corruption-layer.md):** Translate models
2. **[Timeout](../operations/timeouts.md):** Prevent blocking
3. **[Circuit Breaker](../operations/circuit-breaker.md):** Fail fast
4. **[Observability](../operations/observability-basics.md):** Monitor integration health

**Example:** Integrate legacy ERP
- ACL translates flat CSV → domain objects
- Circuit breaker if ERP slow (>5s)
- Fallback: serve cached data

---

### E. Platform / Service-to-Service

#### Dynamic Cluster (Autoscaling, Kubernetes)
**Goal:** No hardcoded endpoints

**Patterns:**
1. **[Service Discovery](../operations/service-discovery.md):** DNS or registry
2. **[Observability](../operations/observability-basics.md):** Monitor instance health

**Implementation:** Kubernetes
- Services call by DNS: `http://user-service`
- Kubernetes resolves to available pods
- Readiness probes ensure only healthy pods receive traffic

---

#### True Service Autonomy
**Goal:** Independent deploy, scale, evolve

**Patterns:**
1. **[Database per Service](../architecture/database-per-service.md):** Isolated data
2. **[Outbox](../architecture/outbox-pattern.md):** Event-driven communication
3. **[Saga](../architecture/saga-pattern.md):** Distributed transactions
4. **[CQRS](../architecture/cqrs.md):** Cross-service queries (if needed)

**Result:** Teams own end-to-end (code, DB, deploy, monitor)

---

## Quick Reference: Pattern by Problem

| Problem | Primary Patterns |
|---------|------------------|
| Cascading failure | Timeout + Retry + Circuit Breaker + Bulkhead |
| Abuse / DDoS | Rate Limiting + API Gateway + Timeout |
| Partial outage | Graceful Degradation + Circuit Breaker + Caching |
| Chatty client | API Composition + BFF |
| Multi-service transaction | Saga + Outbox + Idempotency + Reconciliation |
| Cross-service query | API Composition OR CQRS |
| Legacy integration | ACL + Strangler Fig + Circuit Breaker |
| Different client needs | BFF (one per client) + API Gateway |
| Dynamic endpoints | Service Discovery + Health Checks |
| Event reliability | Outbox + Retry + Idempotency + Reconciliation + DLQ |

---

## Checklist: Essential Combinations

### Every sync call
- [ ] Timeout
- [ ] Retry (with backoff)
- [ ] Circuit Breaker
- [ ] Observability

### Every public API
- [ ] API Gateway
- [ ] Rate Limiting
- [ ] Timeout
- [ ] Observability

### Every event-driven flow
- [ ] Outbox
- [ ] Idempotency
- [ ] Retry
- [ ] DLQ
- [ ] Observability

### Every multi-service transaction
- [ ] Saga (orchestrated or choreographed)
- [ ] Outbox
- [ ] Idempotency
- [ ] Timeout/Circuit Breaker for external calls
- [ ] Observability

---

## Anti-Patterns to Avoid

1. **Distributed transactions (2PC):** Use Saga instead
2. **Shared database across services:** Use Database per Service + events
3. **Direct DB access cross-service:** Use API calls or events
4. **Retries without backoff:** Causes thundering herd
5. **Circuit breaker without timeout:** Can't detect slow failures
6. **BFF with business logic:** Keep BFF thin (aggregation only)
7. **API Gateway with domain logic:** Keep gateway infrastructure-only

---

## Related Topics
- [Microservices design basics](../architecture/microservices-design-basics.md)
- [Resilience patterns overview](../operations/reliability-basics.md)
- [Event-driven architecture](../architecture/event-driven-basics.md)
- [Domain-Driven Design](../architecture/domain-driven-design.md)
