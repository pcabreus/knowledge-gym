---
id: bulkheads
title: "Bulkheads"
type: pattern
status: learning
importance: 70
difficulty: medium
tags: [reliability, isolation, concurrency]
canonical: true
aliases: []
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Bulkheads

## TL;DR (BLUF)
- Bulkheads isolate resources so a failure in one component doesn’t sink the whole system.
- Use them to prevent “noisy neighbor” and [Cascading failure](cascading-failure.md).
- Trade-off: capacity fragmentation and higher operational complexity.

## Definition
**What it is:** A resilience pattern that partitions resources (threads, pools, queues, or services) to contain failures within a bounded fault domain.  
**Key terms:** isolation, partitioned pools, fault domain, noisy neighbor, saturation.

## Why it matters
- Prevents one dependency or tenant from exhausting shared resources.
- Keeps critical paths healthy even when non-critical paths fail.

## Scope & Non-goals
**In scope:** per-dependency pools, per-tenant quotas, and isolated worker groups.  
**Out of scope / NOT solved by this:** root-cause recovery (see [Circuit breaker](circuit-breaker.md)) or overload rejection (see [Load shedding](load-shedding.md)).

## Mental model / Intuition
- Like compartments in a ship: flooding in one compartment doesn’t sink the entire vessel.

## Decision rules (When to use / When not to use)
### Use it when
- You have multiple dependencies with different risk profiles.
- Multi-tenant systems suffer from noisy neighbors.
- Shared pools (threads/DB connections) are a frequent source of incidents.

### Avoid it when
- The system is small and shared resources are not a bottleneck.
- You cannot afford idle capacity fragmentation.

## How I would use it (practical)
- **Context:** A service calls two external APIs: one reliable, one flaky.
- **Steps:**
  1) Split worker pools and connection limits per dependency.
  2) Add per-tenant concurrency caps.
  3) Monitor saturation per pool and tune limits.
- **What success looks like:** failures in the flaky dependency don’t affect the reliable one.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** strong isolation, reduced blast radius.
- **Cons / Risks:** idle capacity, more tuning, and possible under-utilization.
### Alternatives
- **Backpressure:** limit overall load when capacity is tight.
- **Load shedding:** reject or degrade low-priority work.
- **How to choose:** if the main risk is cross-impact between components, choose bulkheads.

## Failure modes & Pitfalls
- Poor sizing → one pool starves while another is underused.
- No per-tenant isolation → a single tenant still dominates a pool.
- Over-partitioning → operational overhead with minimal benefit.

## Observability (How to detect issues)
- **Metrics:** per-pool utilization, queue depth per pool, rejection rate by pool.
- **Logs:** pool saturation events with tenant/dependency identifiers.
- **Traces:** show which pool a request used and where it waited.
- **Alerts:** pool saturation > threshold, per-tenant dominance spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Identify critical dependencies and tenants
  - [ ] Partition pools (threads, queues, connections)
  - [ ] Set per-pool limits and alert thresholds
  - [ ] Validate isolation in load tests
- **Security / Compliance notes**
  - Ensure tenant isolation does not leak data in shared fallbacks
- **Performance notes**
  - Measure utilization to avoid wasting capacity
- **Operational notes**
  - Document pool sizing logic and re-tuning playbooks

## Mini example (if applicable)
```text
critical_pool = 50 connections
noncritical_pool = 10 connections
# critical endpoints never compete with non-critical ones
```

## Common anti-patterns
- **Anti-pattern:** “One giant pool for everything.”
  - **Why it’s bad:** a noisy neighbor can exhaust shared capacity.
  - **Better approach:** split pools by dependency or priority.

## Interview readiness
### “Explain it like I’m teaching”
- Bulkheads isolate capacity so one failing or noisy component can’t take down the rest of the system.

### Trap questions (with answers)
1) **Q:** Do bulkheads eliminate the need for circuit breakers?
   - **A:** No. Bulkheads limit blast radius; breakers stop calls to unhealthy dependencies.
2) **Q:** Are bulkheads only for microservices?
   - **A:** No. You can apply them inside a monolith with per-pool limits.
3) **Q:** Is one pool per request path always best?
   - **A:** No. Over-partitioning wastes capacity and adds overhead.

### Quick self-check (5 items)
- [ ] I can define bulkheads precisely.
- [ ] I can state when to use them and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Distributed systems basics](../system-design/distributed-systems-basics.md)
- [Performance basics](../system-design/performance-basics.md)
- [Observability basics](observability-basics.md)

### Related topics
- [Circuit breaker](circuit-breaker.md)
- [Backpressure](../system-design/backpressure.md)
- [Load shedding](load-shedding.md)
- [Timeouts](timeouts.md)

### Compare with
- [Circuit breaker](circuit-breaker.md) — breaker stops unhealthy calls; bulkheads isolate capacity.
- [Backpressure](../system-design/backpressure.md) — backpressure limits overall load; bulkheads isolate by component.

## Notes / Inbox (optional)
- N/A
