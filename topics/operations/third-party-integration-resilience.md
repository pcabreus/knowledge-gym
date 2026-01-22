---
id: third-party-integration-resilience
title: "Third-Party Integration Resilience"
type: skill
status: learning
importance: 85
difficulty: hard
tags: [reliability, integrations, operations]
canonical: true
aliases: []
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Third-Party Integration Resilience

## TL;DR (BLUF)
- Reliable third-party integrations are built from: timeouts, bounded retries with backoff+jitter, circuit breakers, and idempotency.
- Treat transient vs permanent errors differently; route non-retryable work to DLQ / manual remediation.
- Success is measurable: low error rate, controlled latency p99, and no retry storms.

## Definition
**What it is:** Practices and patterns to integrate with external providers while keeping your system correct and stable under provider failures.
**Key terms:** transient errors, permanent errors, rate limit (rate limiting), timeout, retry, circuit breaker, idempotency key.

## Why it matters
- Third-party APIs are a common single point of failure.
- “It works in dev” breaks in prod due to rate limits, partial outages, and timeouts.

## Scope & Non-goals
**In scope:** outbound HTTP/service calls, inbound webhooks, and async pipelines.
**Out of scope / NOT solved by this:** choosing providers or legal/compliance details.

## Mental model / Intuition
- Assume the provider will be slow, flaky, and occasionally wrong.
- Your job is to contain blast radius and preserve correctness.

## Decision rules (When to use / When not to use)
### Use these controls when
- The call is on the critical path (user-facing latency).
- The call is high volume or has rate limits.

### Avoid complexity when
- The integration is low volume, best-effort, and duplicates/loss are acceptable.

## How I would use it (practical)
- **Context:** Integrating with a voice provider and a payments provider.
- **Steps:**
  1) Add strict [Timeouts](timeouts.md).
  2) Add [Retries](retries-and-backoff.md) only for transient errors and only with a time budget.
  3) Add [Circuit breaker](circuit-breaker.md) with half-open probing.
  4) Use [Idempotency](idempotency.md): idempotency keys, dedup store, and unique constraints.
  5) For async workflows, route permanent failures to [DLQ & poison messages](dlq-and-poison-messages.md).
- **What success looks like:** provider outages degrade gracefully and do not cause cascading failures.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** stable platform under provider incidents.
- **Cons / Risks:** tuning overhead; more moving parts; risk of failing fast too aggressively.

### Alternatives
- **Async queue decoupling:** remove provider from request path.
- **Provider abstraction layer:** to switch providers (with its own costs).
- **How to choose:** if provider is critical, invest in resilience + observability.

## Failure modes & Pitfalls
- Retrying permanent errors (400, auth, schema) → wasted capacity.
- No idempotency when provider times out after processing → double actions.
- No rate limiting / backpressure → self-inflicted 429 storm.

## Observability (How to detect issues)
- **Metrics:** error rate by provider & error class, p95/p99 latency, retry counts, breaker state.
- **Logs:** correlation id across attempts; sanitized request identifiers.
- **Traces:** external call span with tags (provider, route, attempt).
- **Alerts:** spike in 429/503, breaker open, latency p99 regression.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Error taxonomy: transient vs permanent
  - [ ] Timeout per call and overall budget
  - [ ] Retry policy with jitter and max attempts
  - [ ] Circuit breaker thresholds
  - [ ] Idempotency strategy
  - [ ] Runbook for provider outage

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** “If it fails, retry more.”
  - **Why it’s bad:** turns an external incident into your own outage.
  - **Better approach:** budgeted retries + breaker + async fallback.

## Interview readiness
### “Explain it like I’m teaching”
- For third-party integrations I start with timeouts, add bounded retries with jitter for transient errors, protect with a circuit breaker for sustained failures, and ensure idempotency so retries/timeouts never duplicate side effects.

### Trap questions (with answers)
1) **Q:** If the provider returns 500 sometimes, do you retry indefinitely?
   - **A:** No; bounded retries within a time budget and then degrade/queue/DLQ.
2) **Q:** A timeout means the provider didn’t process the request, right?
   - **A:** Not necessarily; timeouts can happen after the provider processed it. That’s why idempotency matters.
3) **Q:** Are webhooks easier because they’re inbound?
   - **A:** Not automatically; you still need idempotency, signature verification, and DLQ/replay patterns.

### Quick self-check (5 items)
- [ ] I can classify errors correctly.
- [ ] I can design timeouts + retries with budgets.
- [ ] I can explain circuit breaker tuning.
- [ ] I can implement idempotency keys.
- [ ] I can propose observability and alerts.

## Links (NO duplication)
### Prerequisites
- [HTTP](http.md)
- [Idempotency](idempotency.md)

### Related topics
- [Retries, exponential backoff, and jitter](retries-and-backoff.md)
- [Timeouts](timeouts.md)
- [Circuit breaker](circuit-breaker.md)
- [DLQ & poison messages](dlq-and-poison-messages.md)
- [Feature flags](feature-flags.md)

### Compare with
- [Event-driven architecture](../architecture/event-driven-basics.md) — async patterns reduce critical-path dependency on third parties.

## Notes / Inbox (optional)
- N/A
