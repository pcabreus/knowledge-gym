---
id: retry-budgets
title: "Retry Budgets"
type: concept
status: learning
importance: 70
difficulty: medium
tags: [reliability, retries, performance]
canonical: true
aliases: []
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Retry Budgets

## TL;DR (BLUF)
- A retry budget caps how much additional load retries are allowed to introduce.
- Use it to prevent retry storms and protect downstream capacity.
- Trade-off: fewer retries can reduce success rate during transient failures.

## Definition
**What it is:** A policy that limits retry volume by allocating a bounded “budget” (percentage or fixed rate) relative to successful requests over a time window.  
**Key terms:** retry ratio, budget window, success rate, retry storm, load amplification.

## Why it matters
- Retries are extra load; without a budget they can amplify failures and trigger [Cascading failure](cascading-failure.md).
- Budgets force you to balance recovery with system stability.

## Scope & Non-goals
**In scope:** client-side retry limits for HTTP calls, queues, and consumers.  
**Out of scope / NOT solved by this:** determining retryable errors (see [Retries, exponential backoff, and jitter](retries-and-backoff.md)) or overload protection alone (see [Backpressure](../system-design/backpressure.md)).

## Mental model / Intuition
- “You can only spend a small percentage of your success capacity on retries.”

## Decision rules (When to use / When not to use)
### Use it when
- A dependency is shared across many clients or tenants.
- You see retry spikes or high tail latency during partial outages.
- You need guardrails to keep retries from amplifying load.

### Avoid it when
- The system is small and retries are rare (still keep a cap).
- You already enforce strict concurrency limits and do not retry (rare).

## How I would use it (practical)
- **Context:** A service calls a payment provider with intermittent timeouts.
- **Steps:**
  1) Use a local token bucket: for every 10 successful requests, earn 1 retry token.
  2) If a request fails, spend 1 token to retry; if no tokens, fail fast.
  3) Combine with per-attempt timeouts and exponential backoff.
- **What success looks like:** retries help transient errors without saturating the dependency.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** prevents retry storms, improves system stability.
- **Cons / Risks:** lower success rate under short transient errors if budget is too strict.
### Alternatives
- **Circuit breaker:** stop calls during sustained failure.
- **Backpressure / load shedding:** reduce upstream load instead of retrying.
- **How to choose:** use budgets when retries are needed but must be bounded; use breakers when dependency is unhealthy.

## Failure modes & Pitfalls
- Budget too high → retries still overwhelm dependencies.
- Budget too low → unnecessary failures during transient issues.
- Ignoring per-tenant limits → one tenant burns the budget.

## Observability (How to detect issues)
- **Metrics:** retry rate, retry budget usage %, success-after-retry %, p95/p99 latency.
- **Logs:** budget decision (allowed/denied), current budget usage.
- **Traces:** tag retries with budget state.
- **Alerts:** budget saturation + error spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define a time window (e.g., 1 minute)
  - [ ] Define a budget (e.g., retries ≤ 10% of successes)
  - [ ] Enforce budget before retrying
  - [ ] Add per-tenant caps if multi-tenant
- **Security / Compliance notes**
  - N/A
- **Performance notes**
  - Keep budget checks low-latency and local if possible
- **Operational notes**
  - Tune budgets based on SLOs and incident history

## Mini example (if applicable)
```text
retry_tokens += floor(successes / 10)
if retry_tokens > 0: retry_tokens -= 1; retry
else: fail_fast
```

## Common anti-patterns
- **Anti-pattern:** “Retries are free, just do more.”
  - **Why it’s bad:** retries add load and can cause outages.
  - **Better approach:** use a budget and backoff.

## Interview readiness
### “Explain it like I’m teaching”
- A retry budget is a cap on retries relative to successful traffic, so retries help recovery without overwhelming dependencies.

### Trap questions (with answers)
1) **Q:** Are retry budgets only for HTTP clients?
   - **A:** No. They apply to queues and consumers too, anywhere retries add load.
2) **Q:** Does a retry budget replace timeouts?
   - **A:** No. Timeouts bound each attempt; budgets bound total retry load.
3) **Q:** Should budgets be global or per-tenant?
   - **A:** Often per-tenant or per-route to prevent one tenant from exhausting the budget.

### Quick self-check (5 items)
- [ ] I can define retry budgets precisely.
- [ ] I can state when to use them and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Timeouts](timeouts.md)
- [Retries, exponential backoff, and jitter](retries-and-backoff.md)

### Related topics
- [Circuit breaker](circuit-breaker.md)
- [Backpressure](../system-design/backpressure.md)
- [Load shedding](load-shedding.md)
- [Cascading failure](cascading-failure.md)
- [Request hedging](request-hedging.md)

### Compare with
- [Circuit breaker](circuit-breaker.md) — breaker stops calling; retry budgets still allow limited retries.
- [Backpressure](../system-design/backpressure.md) — backpressure limits load; budgets limit retry load.

## Notes / Inbox (optional)
- N/A
