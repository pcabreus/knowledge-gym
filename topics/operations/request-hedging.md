---
id: request-hedging
title: "Request Hedging"
type: pattern
status: learning
importance: 65
difficulty: medium
tags: [reliability, performance, latency]
canonical: true
aliases: []
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Request Hedging

## TL;DR (BLUF)
- Request hedging sends a second request if the first is slow to cut tail latency.
- Use it only for idempotent reads and within retry budgets.
- Trade-off: it increases load and can amplify outages if unbounded.

## Definition
**What it is:** A latency-optimization pattern where a duplicate request is issued after a threshold (often p95) and the fastest response wins.  
**Key terms:** tail latency, hedged request, p95 threshold, cancellation, idempotent.

## Why it matters
- Tail latency (p99) often dominates user experience.
- Hedging can reduce tail latency without waiting for slow nodes.

## Scope & Non-goals
**In scope:** read-only, idempotent operations with bounded hedges.  
**Out of scope / NOT solved by this:** write operations or non-idempotent actions (see [Idempotency](idempotency.md)).

## Mental model / Intuition
- “Bet on two horses” when the first one is likely to be slow.

## Decision rules (When to use / When not to use)
### Use it when
- Tail latency is the dominant user pain.
- Operations are read-only or idempotent.
- You can control extra load with [Retry budgets](retry-budgets.md).

### Avoid it when
- The operation is not idempotent or is costly.
- The dependency is already saturated.

## How I would use it (practical)
- **Context:** A read-heavy API with occasional slow nodes.
- **Steps:**
  1) Send request A.
  2) If A exceeds p95 (e.g., 60ms), send request B.
  3) Use the first response, cancel the other.
  4) Enforce hedging within retry budget limits.
- **What success looks like:** p99 latency drops without increasing error rates or saturation.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** better tail latency.
- **Cons / Risks:** extra load, risk of overload amplification.
### Alternatives
- **Timeouts + retries:** for transient failures, not tail latency.
- **Caching:** avoid repeated slow calls.
- **How to choose:** hedge only when tail latency is the dominant issue and you can afford extra load.

## Failure modes & Pitfalls
- Hedging on every request → overload.
- Using hedging on writes → duplicate side effects.
- No cancellation → doubles load unnecessarily.

## Observability (How to detect issues)
- **Metrics:** hedge rate, extra load %, p95/p99 latency, cancellation rate.
- **Logs:** hedge trigger reason and threshold.
- **Traces:** parallel spans and winner selection.
- **Alerts:** hedge rate spike + saturation increase.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Set hedge threshold (e.g., p95)
  - [ ] Restrict to idempotent reads
  - [ ] Enforce retry budget limits
  - [ ] Cancel losing requests
- **Security / Compliance notes**
  - N/A
- **Performance notes**
  - Track incremental load from hedges
- **Operational notes**
  - Tune threshold to avoid excessive hedging

## Mini example (if applicable)
```text
send A
if A > p95_threshold: send B
use first response; cancel the other
```

## Common anti-patterns
- **Anti-pattern:** “Hedge all requests.”
  - **Why it’s bad:** doubles load and can trigger outages.
  - **Better approach:** hedge selectively with a budget.

## Interview readiness
### “Explain it like I’m teaching”
- Request hedging issues a backup request after a latency threshold to reduce tail latency, but it must be bounded and only used for idempotent reads.

### Trap questions (with answers)
1) **Q:** Can you hedge write requests?
   - **A:** No. Unless they are strictly idempotent, you risk duplicate side effects.
2) **Q:** Does hedging replace retries?
   - **A:** No. Hedging targets tail latency; retries target transient failure recovery.
3) **Q:** Is hedging safe without budgets?
   - **A:** No. It can amplify load and cause overload.

### Quick self-check (5 items)
- [ ] I can define request hedging precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Performance basics](../system-design/performance-basics.md)
- [Idempotency](idempotency.md)

### Related topics
- [Retry budgets](retry-budgets.md)
- [Timeouts](timeouts.md)
- [Backpressure](../system-design/backpressure.md)

### Compare with
- [Retries, exponential backoff, and jitter](retries-and-backoff.md) — retries recover failures; hedging reduces tail latency.
- [Caching fundamentals](../system-design/caching-fundamentals.md) — caching reduces calls; hedging duplicates calls.

## Notes / Inbox (optional)
- N/A
