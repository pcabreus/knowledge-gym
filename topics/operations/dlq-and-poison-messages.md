---
id: dlq-and-poison-messages
title: "DLQs and Poison Messages"
type: concept
status: learning
importance: 80
difficulty: medium
tags: [messaging, reliability, operations]
canonical: true
aliases: []
created_at: 2026-01-22
updated_at: 2026-01-22
---

# DLQs and Poison Messages

## TL;DR (BLUF)
- A poison message is one that will *never* succeed with retries.
- A DLQ (dead-letter queue) isolates poison messages so the main pipeline keeps flowing.
- DLQs are an *operational tool*: you need policies for replay, RCA, and prevention.

## Definition
**What it is:** A dead-letter queue stores messages that failed processing after retries or were classified as non-retryable.
**Key terms:** poison message, DLQ, retry policy, quarantine, replay.

## Why it matters
- Without DLQ, poison messages can create infinite retry loops and halt progress.
- DLQ discipline is a major differentiator in senior interviews.

## Scope & Non-goals
**In scope:** classification of errors, DLQ routing, replay process.
**Out of scope / NOT solved by this:** message ordering guarantees.

## Mental model / Intuition
- “Retries are for transient failures; DLQ is for everything else.”

## Decision rules (When to use / When not to use)
### Use a DLQ when
- You have at-least-once processing and retries.
- A single bad message can block a partition/worker.

### Avoid DLQ misuse when
- You treat DLQ as a trash can (no monitoring, no replay plan).

## How I would use it (practical)
- **Context:** Consumer processes events and writes to Postgres.
- **Steps:**
  1) Define retryable vs non-retryable error classes.
  2) Retry transient errors with backoff.
  3) Send non-retryable or max-attempt messages to DLQ with context.
  4) Run a DLQ workflow: alert → triage → fix → replay.
- **What success looks like:** DLQ stays near zero; replays are safe because consumers are idempotent.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** prevents pipeline blockage, enables controlled recovery.
- **Cons / Risks:** operational overhead; replay can reintroduce load if not throttled.

### Alternatives
- **Skip + log:** only if loss is acceptable.
- **Park-and-continue table:** DLQ implemented as a DB table with tooling.
- **How to choose:** prefer DLQ whenever correctness matters and messages can be poison.

## Failure modes & Pitfalls
- DLQ without alerting → silent data loss.
- Replaying without [Idempotency](idempotency.md) → duplicated side effects.
- Putting transient failures into DLQ too early → needless manual work.

## Observability (How to detect issues)
- **Metrics:** DLQ depth, DLQ ingress rate, retry rate, max-attempt rate.
- **Logs:** reason for DLQ routing + classification + original message id.
- **Traces:** show last failing span + exception type.
- **Alerts:** DLQ depth > threshold; DLQ ingress spike; poison rate per integration.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Error taxonomy (transient vs permanent)
  - [ ] Retry policy (max attempts, backoff)
  - [ ] DLQ payload includes enough context to debug
  - [ ] Replay tool/process with rate limits

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** “We’ll just increase retries.”
  - **Why it’s bad:** poison messages never succeed; you only delay recovery.
  - **Better approach:** classify errors and route to DLQ with a runbook.

## Interview readiness
### “Explain it like I’m teaching”
- A DLQ quarantines messages that can’t be processed so the system continues; it’s only useful if you have monitoring and a controlled replay/RCA workflow.

### Trap questions (with answers)
1) **Q:** If you have a DLQ, do you still need retries?
   - **A:** Yes; retries handle transient faults so DLQ is reserved for permanent failures.
2) **Q:** Should you replay DLQ messages immediately after fixing the bug?
   - **A:** Not blindly; you need throttling and you must ensure idempotency to avoid side effects.
3) **Q:** Is a poison message always malformed JSON?
   - **A:** No; it can be valid but violates business rules or depends on missing data.

### Quick self-check (5 items)
- [ ] I can define poison message vs transient failure.
- [ ] I can define a retry policy.
- [ ] I can design a replay workflow.
- [ ] I can propose DLQ alerts.
- [ ] I can explain idempotency’s role in replay.

## Links (NO duplication)
### Prerequisites
- [Retries, exponential backoff, and jitter](retries-and-backoff.md)
- [Idempotency](idempotency.md)

### Related topics
- [Event-driven architecture](../architecture/event-driven-basics.md)
- [Messaging basics](../architecture/messaging-basics.md)

### Compare with
- [Message delivery semantics](../architecture/message-delivery-semantics.md) — delivery semantics create duplicates/loss; DLQ handles non-retryable processing failures.

## Notes / Inbox (optional)
- N/A
