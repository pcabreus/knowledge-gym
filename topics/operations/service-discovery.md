---
id: service-discovery
title: "Service Discovery"
type: pattern
status: learning
importance: 70
difficulty: medium
tags: [operations, microservices, distributed-systems, networking]
canonical: true
aliases: [service registry, service mesh discovery]
created_at: 2026-01-26
updated_at: 2026-01-26
---

# Service Discovery

## TL;DR (BLUF)
- Dynamically discover service instances by logical name instead of hardcoded IP:port.
- Use it in containerized/cloud environments with autoscaling and dynamic IPs.
- Trade-off: added complexity and dependency on registry availability.

## Definition
**What it is:** A mechanism for services to locate each other dynamically using a service registry (DNS, Consul, Kubernetes Service, Eureka) that maps logical service names to available instances (IP:port).

**Key terms:** service registry, DNS, health checks, load balancing, service mesh, dynamic discovery, instance registration.

## Why it matters
- Enables elastic scaling: services can add/remove instances without config changes.
- Supports zero-downtime deploys: new instances register, old ones deregister.
- Simplifies service-to-service communication in microservices.
- Essential for cloud-native and containerized environments (Kubernetes, ECS, Cloud Run).

## Scope & Non-goals
**In scope:** service registration, health checks, DNS-based or registry-based discovery, client-side vs server-side load balancing.

**Out of scope / NOT solved by this:**
- API Gateway routing (see [API Gateway](../system-design/api-gateway.md))
- Cross-cutting concerns (auth, retries) → [Sidecar](sidecar.md) or service mesh
- Business logic or data aggregation

## Mental model / Intuition
- Like a phone directory: instead of memorizing phone numbers, you look up "John's Pizza" and get the current number.
- In microservices: instead of `http://192.168.1.10:8080`, you call `http://user-service/api/users`.

## Decision rules (When to use / When not to use)
### Use it when
- Services run in containers (Docker, Kubernetes) with dynamic IPs.
- You need autoscaling (instances come/go frequently).
- Deploying in cloud environments (AWS, GCP, Azure).
- You have many services and manual config is error-prone.

### Avoid it when
- You have a small, static set of services with fixed IPs.
- Services run on VMs with stable endpoints (rare in modern systems).
- Operational overhead of running a registry outweighs benefits.

## How I would use it (practical)
- **Context:** E-commerce platform on Kubernetes with order-service, user-service, payment-service.
- **Steps:**
  1) Deploy services with Kubernetes Service resources:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: user-service
     spec:
       selector:
         app: user
       ports:
       - port: 80
         targetPort: 8080
     ```
  2) Services call each other by DNS name: `http://user-service/api/users`.
  3) Kubernetes DNS resolves `user-service` to available pod IPs.
  4) Add readiness/liveness probes:
     ```yaml
     livenessProbe:
       httpGet:
         path: /health
         port: 8080
       initialDelaySeconds: 10
       periodSeconds: 5
     ```
  5) Enable **client-side load balancing** (round-robin by default) or use service mesh (Istio, Linkerd).
  6) Monitor: service registry health, failed health checks, DNS resolution latency.

## Trade-offs / Costs (and their mitigation)
| Trade-off | Mitigation |
|-----------|-----------|
| Registry becomes single point of failure | Use highly available registry (Kubernetes DNS, Consul cluster); cache instance list locally |
| Stale instance data (health check lag) | Tune health check intervals (5-10s); use short TTLs |
| Increased complexity (registry + health checks) | Use managed solutions (Kubernetes, AWS Cloud Map); automate with IaC |
| DNS caching issues (clients cache stale IPs) | Set low DNS TTLs (1-5s); use client libraries that respect TTL |
| Client-side load balancing overhead | Use service mesh for server-side LB (Istio, Linkerd) |

## Failure modes / Edge cases
1. **Registry outage:** Services can't discover new instances.
   - *Mitigation:* Cache last-known instances locally; use eventually consistent registry.
2. **Unhealthy instances registered:** Failed health checks lag; clients get 503.
   - *Mitigation:* Aggressive health check intervals (3-5s); fast deregistration.
3. **DNS TTL too high:** Clients cache old IPs after scale-down/deploy.
   - *Mitigation:* Set DNS TTL to 1-5 seconds; use connection pooling with periodic refresh.
4. **Split-brain (multi-region):** Different registries in different regions drift.
   - *Mitigation:* Use regional registries with cross-region replication (eventual consistency).
5. **Service name collision:** Two teams use same service name.
   - *Mitigation:* Namespace services (e.g., `team-a.user-service`).

## Alternatives
- **Hardcoded IPs/config files:** Simple but brittle; requires manual updates.
- **Load balancer with static backend pool:** Works for few services; doesn't scale.
- **Service mesh (Istio, Linkerd):** Combines discovery + mTLS + retries + observability.
- **DNS SRV records:** Lightweight discovery for simple cases.

## Patterns
### 1. Client-side discovery
- Client queries registry, gets instance list, and load-balances.
- **Pros:** No extra hop; client controls LB strategy.
- **Cons:** Client complexity; each language needs library.

### 2. Server-side discovery
- Client calls load balancer (or API Gateway); load balancer queries registry.
- **Pros:** Simple client; centralized LB logic.
- **Cons:** Extra hop; load balancer can be bottleneck.

### 3. Service mesh (Istio, Linkerd)
- Sidecar proxies handle discovery + LB + retries + mTLS.
- **Pros:** Zero code change; consistent policies.
- **Cons:** Operational complexity; added latency.

**Recommendation:** Use **Kubernetes DNS** (simplest) or **service mesh** (if you need mTLS/retries/observability).

## Combinations
Service Discovery is **almost always used with:**
- **[Microservices design](../architecture/microservices-design-basics.md):** Essential for service-to-service calls.
- **[Health checks](../operations/reliability-basics.md):** Ensure only healthy instances are discovered.
- **[Sidecar](sidecar.md):** Service mesh proxies handle discovery + LB.
- **[Observability](../operations/observability-basics.md):** Track service topology, instance health.
- **[Circuit Breaker](circuit-breaker.md):** Protect against unhealthy instances.

**Typical combination:**
- **Kubernetes cluster:** Service Discovery (DNS) + Health checks + Sidecar (Istio/Linkerd) + Observability

## Prerequisites
- Understanding of [Networking basics](../operations/networking-basics.md) and [DNS](../operations/dns.md).
- Familiarity with [Microservices design](../architecture/microservices-design-basics.md).
- Knowledge of containerization (Docker, Kubernetes).

## Related topics
- [API Gateway](../system-design/api-gateway.md): North-south traffic routing.
- [Sidecar](sidecar.md): Service mesh for discovery + mTLS + retries.
- [Load balancing](../system-design/api-design-basics.md): Client-side vs server-side.
- [Observability](../operations/observability-basics.md): Monitor service topology.
- [DNS](../operations/dns.md): Foundation for discovery.

## Real-world examples
1. **Kubernetes:** Built-in DNS-based service discovery; each Service gets a DNS name.
2. **Netflix Eureka:** Client-side discovery registry used in Spring Cloud.
3. **HashiCorp Consul:** Multi-DC service registry with health checks and KV store.
4. **AWS Cloud Map:** Managed service discovery for ECS and EKS.

## Checklist (self-test)
- [ ] I understand client-side vs server-side discovery.
- [ ] I can configure health checks for service instances.
- [ ] I know how DNS TTLs affect discovery staleness.
- [ ] I can explain when to use service mesh vs native Kubernetes DNS.
- [ ] I can monitor service registry health and instance availability.

## Reminders / Key takeaways
- Service discovery is **essential for dynamic environments** (containers, autoscaling).
- Always use **health checks** to avoid routing to unhealthy instances.
- Prefer **Kubernetes DNS** for simplicity, **service mesh** for advanced features.
- Cache instance lists locally to survive registry outages.

## Trap questions (with answers)
### Q1: Can I use service discovery for external APIs (third-party services)?
**A:** **No**. Service discovery is for **internal service-to-service** communication within your cluster. For external APIs, use hardcoded URLs or external DNS. Service discovery assumes you control instance lifecycle and health checks.

### Q2: What's the difference between service discovery and an API Gateway?
**A:** **Service discovery** is for **east-west traffic** (service→service within cluster). **[API Gateway](../system-design/api-gateway.md)** is for **north-south traffic** (client→service from outside cluster). Discovery handles dynamic IPs; gateway handles routing, auth, rate limiting.

### Q3: Should I use client-side or server-side discovery?
**A:** **Depends on platform**. For **Kubernetes**, use server-side (Kubernetes Service + DNS) for simplicity. For **non-containerized** or **polyglot** environments, use client-side (Consul, Eureka) for flexibility. Service mesh (Istio) combines both: sidecar handles discovery (client-side) but is transparent to app code.

### Q4: What happens if the service registry goes down?
**A:** **Clients can't discover new instances**, but existing connections may work. Mitigation: 
1. Cache last-known instances locally.
2. Use **highly available registry** (Kubernetes DNS is distributed; Consul runs in cluster).
3. Set long cache TTLs as fallback (trade staleness for availability).

### Q5: How do I handle service versioning with discovery?
**A:** Use **service name + version** in discovery (e.g., `user-service-v1`, `user-service-v2`) or **metadata tags** (Consul). Clients specify which version to call. Alternatively, use [API Gateway](../system-design/api-gateway.md) for version routing and keep discovery version-agnostic. See [API versioning](../system-design/versioning-apis-and-events.md).
