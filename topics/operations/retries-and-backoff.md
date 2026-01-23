---
id: retries-and-backoff
title: "Retries, Exponential Backoff, and Jitter"
type: concept
status: learning
importance: 80
difficulty: medium
tags: [reliability, integrations, networking]
canonical: true
aliases: []
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Retries, Exponential Backoff, and Jitter

## TL;DR (BLUF)
- Retries are a reliability tool, but uncontrolled retries cause outages.
- Use **bounded retries** with **exponential backoff** and **jitter (jitter)**.
- Only retry when the operation is safe (idempotent) and the error is transient.

## Definition
**What it is:** A strategy to re-attempt failed operations with delays that grow over time.
**Key terms:** exponential backoff, jitter, [Retry budgets](retry-budgets.md), transient error, retry storm.

## Why it matters
- Transient failures (network blips, overload) are normal; retries can recover.
- Bad retries create thundering herds and [Cascading failure](cascading-failure.md).

## Scope & Non-goals
**In scope:** client-side retries for HTTP and message processing.
**Out of scope / NOT solved by this:** protecting an overloaded dependency (see [Circuit breaker](circuit-breaker.md)).

## Mental model / Intuition
- “Retry is load.” Backoff controls how much extra load you add while a system is unhealthy.

## Decision rules (When to use / When not to use)
### Use retries when
- The error is likely transient (timeouts, 429/503, connection resets).
- The operation is idempotent (see [Idempotency](idempotency.md)).

### Avoid retries when
- The error is permanent (400/validation, auth failures, schema mismatch).
- Retrying would amplify harm (payment capture without idempotency).

## How I would use it (practical)
- **Context:** Calling a third-party API.
- **Steps:**
  1) Set a strict timeout per attempt.
  2) Retry 2–4 times with exponential backoff + jitter.
  3) Stop early if the dependency is clearly down (circuit breaker).
- **What success looks like:** transient issues recover; incidents don’t become retry storms.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** higher success rate under transient faults.
- **Cons / Risks:** higher tail latency, extra load, duplicated work if not idempotent.

### Alternatives
- **Queue + async processing:** decouple user latency from transient faults (see [Event-driven architecture](../architecture/event-driven-basics.md)).
- **Fail fast + graceful degradation:** when partial functionality is acceptable.
- **How to choose:** synchronous user flows often need strict budgets; async flows can retry longer with DLQ.

## Failure modes & Pitfalls
- Retrying without timeouts (requests pile up).
- Retrying all errors uniformly (permanent errors waste capacity).
- Coordinated retries without jitter (retry spikes).

## Observability (How to detect issues)
- **Metrics:** retry count per call, success-after-retry %, latency p95/p99, 429/503 rate.
- **Logs:** include attempt number, backoff delay, error class.
- **Traces:** show retry spans and total time budget.
- **Alerts:** retry spike + error spike + saturation signals.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define retryable error classes
  - [ ] Bound retries and total time budget
  - [ ] Add jitter
  - [ ] Ensure idempotency or idempotency keys
  - [ ] Add circuit breaker for sustained failures

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** “Retry forever until it succeeds.”
  - **Why it’s bad:** creates infinite queues and outages.
  - **Better approach:** bounded retries + DLQ + human/automated remediation.

## Interview readiness
### “Explain it like I’m teaching”
- Retries should be bounded and spread out with backoff+jitter; you only retry transient errors and only when the operation is safe.

### Trap questions (with answers)
1) **Q:** Is exponential backoff always better than constant delay?
   - **A:** Usually for avoiding retry storms, but you still need to fit within a time budget.
2) **Q:** Do retries replace the need for circuit breakers?
   - **A:** No; retries handle transient faults, circuit breakers protect during sustained faults.
3) **Q:** Why add jitter?
   - **A:** To avoid synchronized retries across many clients.

### Quick self-check (5 items)
- [ ] I can classify transient vs permanent errors.
- [ ] I can design retries with a time budget.
- [ ] I can explain jitter.
- [ ] I can tie retries to idempotency.
- [ ] I can propose monitoring for retry storms.

## Links (NO duplication)
### Prerequisites
- [Timeouts](timeouts.md)
- [Idempotency](idempotency.md)

### Related topics
- [Circuit breaker](circuit-breaker.md)
- [DLQ & poison messages](dlq-and-poison-messages.md)
- [HTTP](http.md)
- [Retry budgets](retry-budgets.md)
- [Request hedging](request-hedging.md)

### Compare with
- [Circuit breaker](circuit-breaker.md) — backoff retries still call the dependency; a breaker stops calling it.

## Notes / Inbox (optional)
- N/A
