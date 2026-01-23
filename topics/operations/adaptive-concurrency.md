---
id: adaptive-concurrency
title: "Adaptive Concurrency"
type: pattern
status: learning
importance: 65
difficulty: medium
tags: [reliability, performance, concurrency]
canonical: true
aliases: []
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Adaptive Concurrency

## TL;DR (BLUF)
- Adaptive concurrency adjusts in-flight limits based on observed latency, instead of fixed caps.
- Use it when static bulkhead/thread-pool limits are hard to tune.
- Trade-off: it can oscillate if signals are noisy or delayed.

## Definition
**What it is:** A feedback-control approach that dynamically raises or lowers concurrency based on signals like RTT/latency to keep load within a safe operating range.  
**Key terms:** feedback loop, RTT, concurrency limit, saturation, stability.

## Why it matters
- Static limits can underutilize capacity or overload a degraded dependency.
- Adaptive control prevents self-inflicted overload during partial failures.

## Scope & Non-goals
**In scope:** client-side concurrency control for HTTP/RPC/DB dependencies.  
**Out of scope / NOT solved by this:** retry policies (see [Retry budgets](retry-budgets.md)) or dependency health detection (see [Circuit breaker](circuit-breaker.md)).

## Mental model / Intuition
- “Feel the pressure”: when latency rises, back off; when latency drops, cautiously increase.

## Decision rules (When to use / When not to use)
### Use it when
- You cannot reliably set a safe static concurrency limit.
- Latency is a good proxy for saturation.

### Avoid it when
- Latency signals are noisy or dominated by unrelated factors.
- You need strict fixed limits for compliance or quotas.

## How I would use it (practical)
- **Context:** A service calls a dependency with varying capacity across time.
- **Steps:**
  1) Track rolling RTT/latency (e.g., p95).
  2) Increase concurrency when latency is below target; decrease when above.
  3) Enforce min/max bounds to avoid runaway.
- **What success looks like:** stable latency with high utilization and fewer overload spikes.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** better utilization, auto-protection under degradation.
- **Cons / Risks:** oscillations, tuning complexity, sensitivity to noisy signals.
### Alternatives
- **Static bulkheads:** fixed pools by dependency or tenant.
- **Rate limiting:** cap input regardless of latency.
- **How to choose:** if capacity shifts often, prefer adaptive; otherwise use static caps.

## Failure modes & Pitfalls
- Feedback loop too aggressive → oscillations.
- RTT spikes from unrelated causes → unnecessary throttling.
- Missing hard caps → runaway concurrency.

## Observability (How to detect issues)
- **Metrics:** concurrency limit over time, RTT p95, saturation, error rate.
- **Logs:** limit adjustments with reason.
- **Traces:** correlation of latency spikes with limit changes.
- **Alerts:** sustained oscillations or limit pegged at min/max.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define target latency window (e.g., p95)
  - [ ] Choose adjustment step sizes
  - [ ] Enforce min/max concurrency bounds
  - [ ] Use smoothing to reduce noise
- **Security / Compliance notes**
  - N/A
- **Performance notes**
  - Use low-overhead latency measurements
- **Operational notes**
  - Document tuning parameters and rollback

## Mini example (if applicable)
```text
if rtt_p95 > target: limit = max(min_limit, limit - step)
else: limit = min(max_limit, limit + step)
```

## Common anti-patterns
- **Anti-pattern:** “No bounds, let it learn forever.”
  - **Why it’s bad:** can overshoot and overload dependencies.
  - **Better approach:** enforce min/max caps and dampening.

## Interview readiness
### “Explain it like I’m teaching”
- Adaptive concurrency changes the number of in-flight requests based on latency signals, so the system self-protects without static tuning.

### Trap questions (with answers)
1) **Q:** Is adaptive concurrency a replacement for bulkheads?
   - **A:** No. It can complement bulkheads but still needs hard isolation for noisy neighbors.
2) **Q:** Does it remove the need for timeouts?
   - **A:** No. Timeouts are still required per attempt.
3) **Q:** Can it use error rate instead of latency?
   - **A:** It can, but latency is often a faster saturation signal.

### Quick self-check (5 items)
- [ ] I can define adaptive concurrency precisely.
- [ ] I can explain why latency is the control signal.
- [ ] I can name at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Performance basics](../system-design/performance-basics.md)
- [Observability basics](observability-basics.md)

### Related topics
- [Bulkheads](bulkheads.md)
- [Backpressure](../system-design/backpressure.md)
- [Retry budgets](retry-budgets.md)
- [Circuit breaker](circuit-breaker.md)

### Compare with
- [Bulkheads](bulkheads.md) — bulkheads isolate capacity; adaptive concurrency tunes capacity.
- [Backpressure](../system-design/backpressure.md) — backpressure limits load; adaptive concurrency adjusts limits.

## Notes / Inbox (optional)
- N/A
