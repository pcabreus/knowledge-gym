---
id: timeouts
title: "Timeouts and Time Budgets"
type: concept
status: learning
importance: 85
difficulty: medium
tags: [reliability, integrations, distributed-systems]
canonical: true
aliases: []
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Timeouts and Time Budgets

## TL;DR (BLUF)
- Timeouts prevent stuck work and limit blast radius.
- Use a **time budget** for the whole operation and allocate per hop.
- Retries without timeouts are a common production killer.

## Definition
**What it is:** A maximum time allowed for an operation (request, DB call, message processing) before it is canceled/failed.
**Key terms:** timeout, deadline (deadline), time budget, cancellation.

## Why it matters
- Without timeouts, latency and resource usage grow unbounded under failure.
- Timeouts enable predictable tail latency and prevent cascading failures.

## Scope & Non-goals
**In scope:** timeouts for HTTP calls, DB calls, message processing.
**Out of scope / NOT solved by this:** correctness under retries (see [Idempotency](idempotency.md)).

## Mental model / Intuition
- Every request has a “stopwatch”. If it runs out, you must stop work and free resources.

## Decision rules (When to use / When not to use)
### Use timeouts when
- Any operation crosses a network boundary.
- Work uses shared resources (threads, DB connections, queue workers).

### Avoid overly aggressive timeouts when
- You would create false failures for known-long operations; prefer async processing.

## How I would use it (practical)
- **Context:** API call triggers 2 downstream services + DB write.
- **Steps:**
  1) Define end-to-end SLO budget (e.g., 800ms).
  2) Allocate per hop (e.g., 150ms + 150ms + 200ms + overhead).
  3) Ensure retries fit in remaining budget.
- **What success looks like:** bounded p99 and no stuck worker pools.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** bounded tail latency, predictable capacity.
- **Cons / Risks:** false timeouts cause retries and load; tuning is required.

### Alternatives
- **Async with queue:** move long work off the request path.
- **Graceful degradation:** return partial results or “try again later”.
- **How to choose:** if user waits, keep budgets tight; if not, use async.

## Failure modes & Pitfalls
- No timeout at all (default infinite).
- Same timeout for all calls (ignores dependencies).
- Timeouts without cancellation propagation → wasted work continues.

## Observability (How to detect issues)
- **Metrics:** timeout rate, latency percentiles, pool saturation, retries triggered by timeouts.
- **Logs:** “deadline exceeded” with remaining budget and dependency.
- **Traces:** show where time is spent before timeout.
- **Alerts:** sustained timeout-rate increase + saturation.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define end-to-end time budget
  - [ ] Set per-hop timeouts
  - [ ] Propagate cancellation/deadlines across calls
  - [ ] Ensure retries fit within budget

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Increase timeouts to “fix” failures.
  - **Why it’s bad:** hides overload and makes incidents worse.
  - **Better approach:** add backpressure, retries with budgets, and breakers.

## Interview readiness
### “Explain it like I’m teaching”
- Timeouts are a protective boundary: they stop stuck work, limit cascading failure, and force you to allocate an explicit latency budget.

### Trap questions (with answers)
1) **Q:** If you add retries, should you increase timeouts?
   - **A:** Not automatically; retries must fit a budget, otherwise you increase tail latency and load.
2) **Q:** Are timeouts only for HTTP?
   - **A:** No; DB calls, queue processing, and any resource-bounded operation needs them.
3) **Q:** Is a timeout enough without cancellation?
   - **A:** No; if work continues after timeout, you still burn resources and can duplicate side effects.

### Quick self-check (5 items)
- [ ] I can explain why timeouts prevent cascading failures.
- [ ] I can define a time budget and allocate it.
- [ ] I can connect timeouts with retries.
- [ ] I can propose metrics for timeouts.
- [ ] I can describe cancellation propagation.

## Links (NO duplication)
### Prerequisites
- [Distributed systems basics](../system-design/distributed-systems-basics.md)

### Related topics
- [Retries, exponential backoff, and jitter](retries-and-backoff.md)
- [Circuit breaker](circuit-breaker.md)
- [HTTP](http.md)

### Compare with
- [Retries, exponential backoff, and jitter](retries-and-backoff.md) — retries increase time; budgets constrain retries.

## Notes / Inbox (optional)
- N/A
