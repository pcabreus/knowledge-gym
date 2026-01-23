---
id: load-shedding
title: "Load Shedding"
type: pattern
status: learning
importance: 70
difficulty: medium
tags: [reliability, operations, traffic-management]
canonical: true
aliases: []
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Load Shedding

## TL;DR (BLUF)
- Load shedding intentionally rejects or degrades work when demand exceeds safe capacity.
- Use it to protect latency SLOs and avoid [Cascading failure](cascading-failure.md).
- Trade-off: you drop or degrade requests to keep the system stable.

## Definition
**What it is:** A resilience pattern that actively drops, rejects, or degrades requests under overload to keep the system within safe operating limits.  
**Key terms:** admission control, rejection, graceful degradation, prioritization, overload.

## Why it matters
- Prevents tail-latency spikes and resource exhaustion during traffic surges.
- Creates predictable, controlled failure instead of slow meltdown.

## Scope & Non-goals
**In scope:** explicit rejection policies, priority tiers, and degrade paths during overload.  
**Out of scope / NOT solved by this:** long-term capacity shortages (see [Capacity planning](capacity-planning.md)) or correctness under duplicates (see [Idempotency](idempotency.md)).

## Mental model / Intuition
- Like a nightclub: if it’s full, you stop letting people in instead of letting everyone crowd inside.

## Decision rules (When to use / When not to use)
### Use it when
- Demand spikes are short-lived or unpredictable.
- Latency SLOs are critical and you must protect downstream capacity.
- You can degrade or drop lower-priority work safely.

### Avoid it when
- You must process every request and can tolerate delays (prefer async queueing).
- Overload is constant and sustained (prefer scaling and capacity fixes).

## How I would use it (practical)
- **Context:** A public API sees flash traffic during a marketing campaign.
- **Steps:**
  1) Define overload thresholds (CPU, queue depth, DB pool saturation).
  2) Add admission control with 429/503 and `Retry-After`.
  3) Prefer shedding low-priority or non-critical endpoints.
  4) Provide a degraded response where possible.
- **What success looks like:** p95 latency stays within SLO while rejection rate is bounded and transparent.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** stability, controlled latency, predictable failure.
- **Cons / Risks:** user-visible errors, fairness concerns, and tuning complexity.
### Alternatives
- **Backpressure:** slow or queue instead of rejecting work.
- **Scale out:** add capacity if demand is consistently high.
- **How to choose:** if you can’t safely queue and must protect latency, prefer shedding.

## Failure modes & Pitfalls
- Shedding too late → tail latency still explodes.
- Shedding too aggressively → avoidable revenue or UX loss.
- No prioritization → critical requests dropped with non-critical ones.
- Retrying from clients without budgets → self-inflicted thundering herd.

## Observability (How to detect issues)
- **Metrics:** rejection rate, p95/p99 latency, saturation (CPU, pool usage), queue depth.
- **Logs:** admission-control decisions (threshold, reason, endpoint, tier).
- **Traces:** identify where requests are rejected or degraded.
- **Alerts:** sustained reject rate + saturation beyond thresholds.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define overload thresholds and signals
  - [ ] Implement tiered policies (critical vs non-critical)
  - [ ] Add clear client error responses and `Retry-After`
  - [ ] Ensure retries are bounded and do not amplify load
- **Security / Compliance notes**
  - Avoid leaking restricted data in degraded responses
- **Performance notes**
  - Prefer fast-fail over long queues when latency is the priority
- **Operational notes**
  - Document reject reasons and runbooks

## Mini example (if applicable)
```text
if cpu > 85% or db_pool_saturation > 90%:
  reject non-critical requests with 503 + Retry-After
  serve cached/degraded response for read-only endpoints
```

## Common anti-patterns
- **Anti-pattern:** “Just add a bigger queue.”
  - **Why it’s bad:** it hides overload and pushes failures to the tail.
  - **Better approach:** shed or degrade low-priority work quickly.

## Interview readiness
### “Explain it like I’m teaching”
- Load shedding is deliberate rejection or degradation when the system is overloaded, so it stays stable and protects latency SLOs.

### Trap questions (with answers)
1) **Q:** Is load shedding the same as backpressure?
   - **A:** No. Backpressure slows/queues producers; load shedding drops or degrades requests.
2) **Q:** Should you always shed at the edge?
   - **A:** Not only at the edge; internal services may also need local shedding to protect shared pools.
3) **Q:** Does load shedding replace scaling?
   - **A:** No. It handles bursts; sustained demand still requires capacity changes.

### Quick self-check (5 items)
- [ ] I can define load shedding precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Distributed systems basics](../system-design/distributed-systems-basics.md)
- [Performance basics](../system-design/performance-basics.md)
- [Observability basics](observability-basics.md)

### Related topics
- [Backpressure](../system-design/backpressure.md)
- [Circuit breaker](circuit-breaker.md)
- [Bulkheads](bulkheads.md)
- (TODO) Rate limiting
- (TODO) Graceful degradation

### Compare with
- [Backpressure](../system-design/backpressure.md) — backpressure slows/queues; load shedding rejects/degrades.
- [Circuit breaker](circuit-breaker.md) — breaker blocks calls to unhealthy dependencies; shedding protects overall capacity.

## Notes / Inbox (optional)
- N/A
