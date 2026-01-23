---
id: cascading-failure
title: "Cascading Failure"
type: concept
status: learning
importance: 70
difficulty: medium
tags: [reliability, operations, distributed-systems]
canonical: true
aliases: []
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Cascading Failure

## TL;DR (BLUF)
- Cascading failure is when a local failure propagates across dependencies and amplifies into system-wide outage.
- It’s usually driven by coupling, retries without budgets, and shared resource saturation.
- Trade-off: preventing cascades often means rejecting or degrading work to protect stability.

## Definition
**What it is:** A failure propagation pattern where an initial fault or overload in one component causes downstream or upstream components to fail, spreading across the system.  
**Key terms:** failure amplification, dependency chain, retry storm, saturation, tail latency.

## Why it matters
- Cascades turn partial failures into full outages.
- They are a top cause of prolonged incidents in distributed systems.

## Scope & Non-goals
**In scope:** failure propagation through synchronous calls, shared pools, and retry amplification.  
**Out of scope / NOT solved by this:** root-cause fixes (see [Incident management basics](incident-management-basics.md)) or availability targets (see [Availability basics](availability-basics.md)).

## Mental model / Intuition
- One service goes slow → upstream waits → queues grow → pools saturate → everything slows or fails.

## Decision rules (When to use / When not to use)
### Use it when
- You are designing or reviewing multi-service call chains.
- You see tail latency spikes or pool saturation under partial outages.

### Avoid it when
- The system is isolated with no critical dependencies (rare in practice).

## How I would use it (practical)
- **Context:** An API calls two downstream services synchronously and has strict latency SLOs.
- **Steps:**
  1) Add time budgets and per-hop [Timeouts](timeouts.md).
  2) Enforce [Backpressure](../system-design/backpressure.md) and [Load shedding](load-shedding.md).
  3) Isolate with [Bulkheads](bulkheads.md).
  4) Add [Circuit breaker](circuit-breaker.md) to stop unhealthy calls.
- **What success looks like:** partial dependency failures do not collapse the whole request path.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** fewer systemic outages, controlled degradation.
- **Cons / Risks:** more tuning, more rejection, and lower peak throughput.
### Alternatives
- **Async decoupling:** move non-critical work off the request path.
- **How to choose:** if synchronous coupling is required, invest in defenses-in-depth.

## Failure modes & Pitfalls
- Unlimited retries without budgets.
- Shared pools across critical and non-critical paths.
- Lack of time budgets and cancellation propagation.

## Observability (How to detect issues)
- **Metrics:** p95/p99 latency, saturation (CPU, pool usage), retry rates, queue depth.
- **Logs:** timeouts, retry counts, fallback activation.
- **Traces:** elongated spans across dependency chains.
- **Alerts:** correlated latency spikes across services + saturation.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define end-to-end budgets and per-hop timeouts
  - [ ] Bound retries and add jitter
  - [ ] Apply backpressure and load shedding
  - [ ] Isolate with bulkheads
  - [ ] Add circuit breakers for unhealthy dependencies
- **Security / Compliance notes**
  - Ensure degraded responses don’t leak sensitive data
- **Performance notes**
  - Prefer bounded queues over unbounded buffers
- **Operational notes**
  - Document fallback and shedding policies

## Mini example (if applicable)
```text
A -> B -> C (sync)
C slows down → B queues grow → A waits → A pool saturates → A starts timing out
```

## Common anti-patterns
- **Anti-pattern:** “Just add more retries.”
  - **Why it’s bad:** retries amplify load and accelerate failure propagation.
  - **Better approach:** add budgets, backpressure, and breakers.

## Interview readiness
### “Explain it like I’m teaching”
- Cascading failure happens when a local fault spreads through dependencies and shared resources, turning partial failures into a system-wide outage.

### Trap questions (with answers)
1) **Q:** Do cascading failures only happen with synchronous calls?
   - **A:** No. Async systems can cascade via queue buildup, retry storms, or shared resource saturation.
2) **Q:** Are retries always good for avoiding cascades?
   - **A:** No. Unbounded retries amplify load and worsen the cascade.
3) **Q:** Does a circuit breaker alone prevent cascades?
   - **A:** Not always. You also need timeouts, backpressure, and isolation.

### Quick self-check (5 items)
- [ ] I can define cascading failure precisely.
- [ ] I can explain how failures propagate.
- [ ] I can name at least 2 defenses.
- [ ] I can describe 1 failure mode.
- [ ] I can propose detection metrics.

## Links (NO duplication)
### Prerequisites
- [Distributed systems basics](../system-design/distributed-systems-basics.md)
- [Performance basics](../system-design/performance-basics.md)
- [Observability basics](observability-basics.md)

### Related topics
- [Timeouts](timeouts.md)
- [Retries, exponential backoff, and jitter](retries-and-backoff.md)
- [Circuit breaker](circuit-breaker.md)
- [Backpressure](../system-design/backpressure.md)
- [Load shedding](load-shedding.md)
- [Bulkheads](bulkheads.md)
- [Request-response architecture](../architecture/request-response.md)
- [Sync vs async communication](../system-design/sync-vs-async-communication.md)

### Compare with
- [Backpressure](../system-design/backpressure.md) — limits load to reduce failure propagation.
- [Load shedding](load-shedding.md) — drops/degrades work to keep capacity stable.

## Notes / Inbox (optional)
- N/A
