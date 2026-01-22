---
id: message-delivery-semantics
title: "Message Delivery Semantics (at-most-once / at-least-once / exactly-once)"
type: concept
status: learning
importance: 85
difficulty: hard
tags: [messaging, kafka, reliability, distributed-systems]
canonical: true
aliases: []
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Message Delivery Semantics (at-most-once / at-least-once / exactly-once)

## TL;DR (BLUF)
- Delivery semantics describe the *probability of loss vs duplication* when moving messages through a system.
- In practice, **at-least-once** is common; you pay for it with **idempotency (idempotency)** on consumers.
- **Exactly-once** is usually scoped (e.g., within a broker) and rarely true end-to-end.

## Definition
**What it is:** A set of guarantees about whether a message is delivered 0, 1, or many times across failures, retries, and restarts.
**Key terms:** at-most-once, at-least-once, exactly-once, duplicate, loss, acknowledgement (ack), offset, replay.

## Why it matters
- It determines which failures you must design for: lost work vs duplicate work.
- Most real incidents in event pipelines are “silent loss” or “duplicate side effects”.

## Scope & Non-goals
**In scope:** semantics of *processing* and *side effects* (DB writes, API calls) under retries.
**Out of scope / NOT solved by this:** ordering guarantees and business-level consistency (see [Event-driven architecture](event-driven-basics.md)).

## Mental model / Intuition
- Think “**what happens when something fails after I did the work but before I recorded that I did the work?**”
- Exactly-once is hard because the *read → process → write* boundary crosses systems.

## Decision rules (When to use / When not to use)
### Use at-most-once when
- Losing some events is acceptable (telemetry sampling, best-effort notifications).
- You value low latency and simplicity over correctness.

### Use at-least-once when
- Losing events is unacceptable, but duplicates are tolerable via [Idempotency](../operations/idempotency.md).
- You can store state to deduplicate or make side effects idempotent.

### Use exactly-once when
- Your platform supports it *for the exact boundary you need* and you can prove it.
- You can keep the workflow inside a transaction boundary (or a supported EOS boundary).

### Avoid “exactly-once” claims when
- You cross a broker + database + third-party API boundary.
- You cannot reason about duplicates during retries or replays.

## How I would use it (practical)
- **Context:** A Kafka consumer writes to Postgres and calls a third-party API.
- **Steps:**
  1) Assume **at-least-once** delivery.
  2) Make the consumer idempotent (dedup key + unique constraint) and treat API calls with idempotency keys.
  3) Use [DLQ & poison messages](../operations/dlq-and-poison-messages.md) for non-retryable failures.
- **What success looks like:** replays/retries do not create extra payments/emails/records.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - At-least-once prevents silent loss.
- **Cons / Risks:**
  - Duplicates are inevitable → you must implement idempotency.
  - Exactly-once support is often partial and operationally complex.

### Alternatives
- **Synchronous request/response:** for strongly consistent user flows (see [Request-response architecture](request-response.md)).
- **Outbox pattern:** for reliable publish after DB commit (see [Outbox pattern](outbox-pattern.md)).
- **How to choose:** if correctness matters, prefer at-least-once + idempotency; treat EOS as an optimization only when scope matches.

## Failure modes & Pitfalls
- Confusing “delivered once to consumer” with “side effect applied once”.
- Acking/committing offsets before side effects → silent loss.
- Retrying non-idempotent operations → duplicated side effects.

## Observability (How to detect issues)
- **Metrics:** duplicate rate (dedup hits), consumer lag, retry rate, DLQ depth.
- **Logs:** dedup decision logs (key, original message id), offset/ack lifecycle.
- **Traces:** spans for read → process → write, with retries annotated.
- **Alerts:** DLQ spikes, sustained dedup hits, lag growth.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Pick a stable message id / idempotency key
  - [ ] Decide ack/commit point relative to side effects
  - [ ] Add dedup store or unique constraints
  - [ ] Define retry vs DLQ policy

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** “We use Kafka, so we’re exactly-once.”
  - **Why it’s bad:** Kafka’s EOS (exactly-once semantics) is scoped; it doesn’t magically make DB writes exactly-once.
  - **Better approach:** design for at-least-once and prove idempotency.

## Interview readiness
### “Explain it like I’m teaching”
- At-most-once risks losing messages, at-least-once risks duplicates, and exactly-once is hard across system boundaries—so most systems pick at-least-once and make consumers idempotent.

### Trap questions (with answers)
1) **Q:** If a consumer commits offsets only after processing, do we have exactly-once?
   - **A:** No; you have at-least-once delivery to processing, but side effects can still duplicate if the process crashes after writing but before committing.
2) **Q:** Can I use retries to “fix” at-most-once loss?
   - **A:** Not reliably; if you drop the message without persistence, there’s nothing to retry.
3) **Q:** Does “exactly-once” remove the need for idempotency?
   - **A:** Usually no; operational edge cases, replays, and cross-system boundaries still make idempotency valuable.

### Quick self-check (5 items)
- [ ] I can define at-most-once vs at-least-once vs exactly-once.
- [ ] I can explain why exactly-once is scoped.
- [ ] I can describe where to ack/commit relative to side effects.
- [ ] I can propose an idempotency strategy.
- [ ] I can name 1 metric to detect duplicates/loss.

## Links (NO duplication)
### Prerequisites
- [Event-driven architecture](event-driven-basics.md)
- [Kafka](kafka.md)
- [Idempotency](../operations/idempotency.md)

### Related topics
- [Messaging basics](messaging-basics.md)
- [Outbox pattern](outbox-pattern.md)
- [DLQ & poison messages](../operations/dlq-and-poison-messages.md)

### Compare with
- [Request-response architecture](request-response.md) — synchronous coupling vs asynchronous delivery semantics.

## Notes / Inbox (optional)
- N/A
