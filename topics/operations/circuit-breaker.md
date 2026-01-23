---
id: circuit-breaker
title: "Circuit Breaker"
type: pattern
status: learning
importance: 75
difficulty: medium
tags: [reliability, integrations, operations]
canonical: true
aliases: []
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Circuit Breaker

## TL;DR (BLUF)
- A circuit breaker stops calling a failing dependency to prevent cascading failure.
- It complements [Timeouts](timeouts.md) and [Retries](retries-and-backoff.md).
- Trade-off: you may fail requests that *could* have succeeded.

## Definition
**What it is:** A resilience pattern that transitions between **closed** (normal calls), **open** (fail fast), and **half-open** (probe) based on recent failures.
**Key terms:** closed/open/half-open, failure threshold, probe, fallback.

## Why it matters
- Prevents retry storms and protects shared resources (threads, DB pools).
- Creates fast, observable failure instead of slow meltdown.

## Scope & Non-goals
**In scope:** client-side breakers for HTTP/DB/third-party calls.
**Out of scope / NOT solved by this:** correctness under duplicates (see [Idempotency](idempotency.md)).

## Mental model / Intuition
- “When something is clearly broken, stop poking it continuously.”

## Decision rules (When to use / When not to use)
### Use it when
- A dependency is flaky or overload-prone and failures cluster.
- Calls are expensive and can saturate your service.

### Avoid it when
- The dependency is in-process and cheap (breaker adds overhead).
- You have no safe fallback and must attempt every call (rare).

## How I would use it (practical)
- **Context:** A contact platform calls a voice provider API.
- **Steps:**
  1) Set strict timeouts.
  2) Add bounded retries.
  3) Add breaker: open on high timeout/5xx rate; half-open probes every N seconds.
  4) Define fallback (queue work, degrade to SMS, or “try later”).
- **What success looks like:** dependency incidents don’t take down the whole platform.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** protects system health, improves tail latency under failure.
- **Cons / Risks:** false opens cause avoidable failures; requires tuning.

### Alternatives
- **[Backpressure](../system-design/backpressure.md) / load shedding:** stop accepting new work under overload.
- **Queue-based async:** remove dependency from request path.
- **How to choose:** if dependency is external and flaky, breaker is usually worth it.

## Failure modes & Pitfalls
- Breaking on the wrong signals (e.g., treating 4xx as dependency failure).
- No half-open probes → breaker stays open too long.
- No per-route/per-tenant isolation → one noisy path opens the breaker for all.

## Observability (How to detect issues)
- **Metrics:** breaker state, open/close transitions, failure rate by dependency.
- **Logs:** state transitions with reason (threshold exceeded).
- **Traces:** annotate fallback path.
- **Alerts:** breaker open for >X minutes + dependency error spike.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Choose signals (timeout %, 5xx %, latency)
  - [ ] Set thresholds and window
  - [ ] Implement half-open probes
  - [ ] Define fallback

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** “Breaker open means the dependency is down.”
  - **Why it’s bad:** the breaker is your *client’s* view; it can be mis-tuned.
  - **Better approach:** correlate breaker state with dependency metrics.

## Interview readiness
### “Explain it like I’m teaching”
- A circuit breaker fails fast when a dependency is unhealthy, preventing your service from wasting resources and cascading failures, and periodically probes to recover.

### Trap questions (with answers)
1) **Q:** Should you use retries *and* a circuit breaker?
   - **A:** Yes; retries help transient faults, breaker handles sustained faults.
2) **Q:** Should 400/validation errors open a breaker?
   - **A:** No; those are usually caller bugs, not dependency health.
3) **Q:** If a breaker opens, do you still need timeouts?
   - **A:** Yes; timeouts are the first line of defense per attempt.

### Quick self-check (5 items)
- [ ] I can explain breaker states.
- [ ] I can choose signals and thresholds.
- [ ] I can describe half-open probing.
- [ ] I can describe fallbacks.
- [ ] I can propose alerts.

## Links (NO duplication)
### Prerequisites
- [Timeouts](timeouts.md)
- [Retries, exponential backoff, and jitter](retries-and-backoff.md)

### Related topics
- [Third-party integration resilience](third-party-integration-resilience.md)

### Compare with
- [Timeouts](timeouts.md) — timeouts bound one call; a breaker governs many calls over time.

## Notes / Inbox (optional)
- N/A
