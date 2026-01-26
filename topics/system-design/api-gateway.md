---
id: api-gateway
title: "API Gateway"
type: pattern
status: learning
importance: 80
difficulty: medium
tags: [architecture, api-design, system-design, microservices]
canonical: true
aliases: []
created_at: 2026-01-26
updated_at: 2026-01-26
---

# API Gateway

## TL;DR (BLUF)
- A single entry point that routes requests to microservices and applies cross-cutting policies.
- Use it for public APIs, multi-client scenarios, and to centralize auth, rate limiting, and observability.
- Trade-off: added latency and a potential single point of failure.

## Definition
**What it is:** A reverse proxy that acts as the single entry point for client requests, routing them to backend microservices while handling cross-cutting concerns like authentication, rate limiting, TLS termination, and request/response transformation.

**Key terms:** routing, reverse proxy, TLS termination, authentication, rate limiting, request aggregation, single entry point.

## Why it matters
- Simplifies client integration by hiding internal service topology.
- Centralizes security policies (auth, CORS, TLS) and operational concerns (logging, tracing, metrics).
- Reduces coupling between clients and services, enabling independent evolution.
- Protects backend services from abuse and overload.

## Scope & Non-goals
**In scope:** routing, auth enforcement, rate limiting, request/response transformation, protocol translation (HTTP→gRPC), TLS termination.

**Out of scope / NOT solved by this:** 
- Business logic (belongs in services or [BFF](../architecture/backend-for-frontend.md))
- Data aggregation for specific clients (see [API Composition](../architecture/api-composition.md) or [BFF](../architecture/backend-for-frontend.md))
- Service mesh concerns within the cluster (see [Sidecar](../operations/sidecar.md))

## Mental model / Intuition
- Think of it as a hotel concierge: one entrance, handles guest identification, directs them to rooms (services), and enforces house rules.
- In microservices: clients call one URL; the gateway knows which service handles which path.

## Decision rules (When to use / When not to use)
### Use it when
- You have multiple microservices behind a public API.
- You need centralized auth, rate limiting, or TLS termination.
- Clients (web, mobile, partners) should not know internal service structure.
- You want to version APIs or perform A/B routing.
- You need request/response transformation or protocol translation.

### Avoid it when
- You have a monolith or single service (adds unnecessary complexity).
- Internal service-to-service calls (use [Service Discovery](../operations/service-discovery.md) instead).
- You need client-specific aggregation logic (use [BFF](../architecture/backend-for-frontend.md) instead).

## How I would use it (practical)
- **Context:** An e-commerce platform with separate services for users, products, orders, and payments exposed to web and mobile clients.
- **Steps:**
  1) Deploy an API Gateway (Kong, AWS API Gateway, or NGINX).
  2) Configure routing rules: `/users/*` → user-service, `/products/*` → product-service.
  3) Enable JWT validation at the gateway for all authenticated endpoints.
  4) Apply rate limiting per client (100 req/min for free users, 1000 for premium).
  5) Enable TLS termination and forward internal traffic as HTTP.
  6) Add distributed tracing headers (X-Request-ID, trace context).
  7) Monitor gateway metrics: request rate, latency p95/p99, error rate by route.

## Trade-offs / Costs (and their mitigation)
| Trade-off | Mitigation |
|-----------|-----------|
| Added latency (~5-20ms) | Deploy gateways close to clients; optimize routing rules |
| Single point of failure | Run multiple gateway instances behind load balancer; use health checks |
| Configuration complexity | Use declarative configs (YAML/JSON); automate with IaC |
| Can become a bottleneck | Scale horizontally; avoid putting business logic in gateway |
| Version sprawl | Enforce deprecation policies; track usage with [Observability](../operations/observability-basics.md) |

## Failure modes / Edge cases
1. **Gateway becomes a bottleneck:** If all traffic flows through one instance, it saturates CPU/network.
   - *Mitigation:* Scale horizontally with load balancer.
2. **Config drift across environments:** Staging vs prod have different routing rules.
   - *Mitigation:* Use GitOps and automated deployments.
3. **Business logic creep:** Teams add transformations/validations in gateway.
   - *Mitigation:* Enforce policy: gateway handles only routing + cross-cutting concerns.
4. **Cascading failures:** If gateway has aggressive [Timeouts](../operations/timeouts.md), it can amplify backend slowness.
   - *Mitigation:* Combine with [Circuit Breaker](../operations/circuit-breaker.md) and [Graceful Degradation](../operations/graceful-degradation.md).
5. **Authentication bypass:** Misconfigured routes allow unauthenticated access.
   - *Mitigation:* Default-deny policy; automated security tests.

## Alternatives
- **Service mesh (Istio, Linkerd):** For internal service-to-service traffic with mTLS, retries, tracing.
- **BFF (Backend for Frontend):** When you need client-specific logic or aggregation.
- **Direct service exposure:** Simpler but exposes internal topology and lacks centralized policies.

## Combinations
API Gateway is **almost always used with:**
- **[Rate Limiting](rate-limiting.md):** Protects backend services from abuse.
- **[Timeout](../operations/timeouts.md):** Prevents hanging requests.
- **[Circuit Breaker](../operations/circuit-breaker.md):** Fails fast when backends are down.
- **[Observability](../operations/observability-basics.md):** Logs, metrics, tracing at the edge.
- **[BFF](../architecture/backend-for-frontend.md):** Gateway routes to BFFs, which aggregate/transform.

**Typical combination:**
- **Public API scenario:** API Gateway + Rate Limiting + Timeout + Retry (with backoff) + Circuit Breaker + Observability

## Prerequisites
- Understanding of [HTTP](../operations/http.md) and reverse proxies.
- Familiarity with [Microservices design](../architecture/microservices-design-basics.md).
- Basic knowledge of [TLS](../operations/tls.md) and authentication (JWT, OAuth).

## Related topics
- [BFF (Backend for Frontend)](../architecture/backend-for-frontend.md): Client-specific aggregation layer.
- [Service Discovery](../operations/service-discovery.md): Dynamic service location.
- [Rate Limiting](rate-limiting.md): Protect from abuse.
- [Observability](../operations/observability-basics.md): Monitor gateway health.
- [API versioning](versioning-apis-and-events.md): Manage API evolution.

## Real-world examples
1. **Netflix Zuul / Spring Cloud Gateway:** Routes requests to hundreds of microservices.
2. **Stripe API Gateway:** Handles auth, rate limiting, versioning for public API.
3. **AWS API Gateway:** Managed service with built-in auth, throttling, caching.

## Checklist (self-test)
- [ ] I understand when to use an API Gateway vs BFF vs Service Mesh.
- [ ] I can design routing rules for multi-service APIs.
- [ ] I know how to apply rate limiting and auth at the gateway.
- [ ] I can explain the failure modes (bottleneck, config drift, logic creep).
- [ ] I can monitor gateway latency, error rate, and throughput.

## Reminders / Key takeaways
- Gateway is for **cross-cutting concerns**, not business logic.
- Always combine with [Timeout](../operations/timeouts.md), [Circuit Breaker](../operations/circuit-breaker.md), and [Observability](../operations/observability-basics.md).
- Scale horizontally to avoid bottleneck.
- Use BFF when you need client-specific aggregation.

## Trap questions (with answers)
### Q1: Can I use an API Gateway for internal service-to-service communication?
**A:** Technically yes, but **not recommended**. API Gateway adds latency and is designed for north-south (client→service) traffic. For east-west (service→service), use [Service Discovery](../operations/service-discovery.md) or a [Service Mesh](../operations/sidecar.md) (Istio, Linkerd) which provides mTLS, retries, and observability without centralized routing.

### Q2: Should I put data aggregation logic in the API Gateway?
**A:** **No**. Aggregation is business logic and belongs in services or a [BFF](../architecture/backend-for-frontend.md). Gateways should only handle routing, auth, rate limiting, and protocol translation. Mixing concerns makes the gateway a monolith and couples it to service internals.

### Q3: If my gateway is highly available, do I still need circuit breakers?
**A:** **Yes**. High availability protects the gateway itself, but not the backends. If a backend service is slow/failing, the gateway can still saturate its connection pools and cascade the failure. Combine with [Circuit Breaker](../operations/circuit-breaker.md) and [Timeout](../operations/timeouts.md) to fail fast.

### Q4: Can an API Gateway replace a load balancer?
**A:** **Partially**. A gateway can route to multiple instances of a service (basic load balancing), but typically you still need a load balancer **in front of the gateway** for HA and scaling. Additionally, Layer 4 load balancers (TCP/UDP) are more efficient for raw traffic distribution than Layer 7 gateways.

### Q5: What's the difference between API Gateway and BFF?
**A:** **API Gateway** is infrastructure: routing, auth, rate limiting for **all clients**. **BFF** is a service: client-specific aggregation and transformation (one BFF per client type: web-BFF, mobile-BFF). You often use both: Gateway routes to BFFs, which aggregate from backend services. See [BFF](../architecture/backend-for-frontend.md).
