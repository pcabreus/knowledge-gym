---
id: idempotency
title: "Idempotency"
type: concept
status: learning
importance: 90
difficulty: hard
tags: [reliability, messaging, http, distributed-systems]
canonical: true
aliases: []
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Idempotency

## TL;DR (BLUF)
- Idempotency means “doing the same request/message twice has the same effect as doing it once”.
- You need it because retries and replays are unavoidable in distributed systems.
- Common tools: idempotency keys, unique constraints, dedup stores, and [Outbox pattern](../architecture/outbox-pattern.md).

## Definition
**What it is:** A property where applying the same operation multiple times produces the same final state and does not duplicate side effects.
**Key terms:** idempotency key (idempotency key), deduplication store (dedup store), replay, side effect, unique constraint.

## Why it matters
- At-least-once delivery is common; without idempotency you get double charges, duplicate emails, or duplicated state.
- It’s one of the highest ROI reliability concepts in interviews and production.

## Scope & Non-goals
**In scope:** idempotency for HTTP handlers and message consumers.
**Out of scope / NOT solved by this:** strict exactly-once end-to-end delivery (see [Message delivery semantics](../architecture/message-delivery-semantics.md)).

## Mental model / Intuition
- Assume every message/request can arrive twice.
- Design the *state transition* so the second application is a no-op.

## Decision rules (When to use / When not to use)
### Use it when
- You retry operations (client retries, queue retries, workflow retries).
- You process messages from brokers/queues with at-least-once delivery.
- You have “money moves”, irreversible actions, or user-visible side effects.

### Avoid / don’t overbuild when
- The operation is naturally idempotent (pure reads) and you can tolerate duplicates.
- The action is best-effort and duplicates are harmless (some notifications).

## How I would use it (practical)
- **Context:** A consumer processes `PaymentRequested` events.
- **Steps:**
  1) Choose a stable key (event id, or business key like `payment_id`).
  2) Store it in a dedup table with a unique constraint.
  3) On duplicate, short-circuit and emit a “duplicate ignored” log.
- **What success looks like:** reprocessing the same event does not create extra payments.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** correctness under retries/replays, simpler incident recovery (safe replays).
- **Cons / Risks:** storage overhead, complexity in key choice, and edge cases with partial failures.

### Alternatives
- **Exactly-once platform features:** only when the boundary matches your need.
- **Manual reconciliation:** sometimes acceptable for analytics pipelines.
- **How to choose:** prefer idempotency as the default; treat EOS as a limited optimization.

## Failure modes & Pitfalls
- Using an unstable key (hash of payload that changes, timestamps).
- Making DB writes idempotent but leaving external API calls non-idempotent.
- Dedup store without TTL/cleanup → unbounded growth.

## Observability (How to detect issues)
- **Metrics:** dedup hit rate, duplicate side-effect rate, retry counts.
- **Logs:** idempotency key + decision (first-seen vs duplicate) + correlation id.
- **Traces:** annotate retries and dedup checks.
- **Alerts:** spikes in dedup hits or in “already exists” constraint violations.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define idempotency key format and scope
  - [ ] Store key with unique constraint (or atomic upsert)
  - [ ] Make the whole handler atomic where possible
  - [ ] Add TTL/cleanup for dedup records
  - [ ] Document retry/duplication behavior in the contract
- **Security / Compliance notes**
  - Don’t log full PII in keys; use stable ids.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** “We’ll just retry until it works.”
  - **Why it’s bad:** without idempotency you eventually succeed twice.
  - **Better approach:** idempotent handlers + bounded retries + DLQ.

## Interview readiness
### “Explain it like I’m teaching”
- Idempotency is designing operations so duplicates don’t create extra side effects; it’s required because distributed systems retry and replay.

### Trap questions (with answers)
1) **Q:** Is HTTP `POST` always non-idempotent?
   - **A:** Semantically it can be non-idempotent, but you can implement idempotency using idempotency keys and server-side dedup.
2) **Q:** Are unique constraints enough for idempotency?
   - **A:** Often they help, but you still must handle external side effects and define what “same operation” means.
3) **Q:** If consumers are idempotent, do we still need a DLQ?
   - **A:** Yes; idempotency handles duplicates, not poison messages or permanent failures.

### Quick self-check (5 items)
- [ ] I can define idempotency precisely.
- [ ] I can propose an idempotency key for a workflow.
- [ ] I can explain dedup stores vs unique constraints.
- [ ] I can handle idempotency with third-party APIs.
- [ ] I can name 1 failure mode and detection metric.

## Links (NO duplication)
### Prerequisites
- [Message delivery semantics](../architecture/message-delivery-semantics.md)
- [HTTP](http.md)
- [Outbox pattern](../architecture/outbox-pattern.md)

### Related topics
- [Event-driven architecture](../architecture/event-driven-basics.md)
- [Kafka](../architecture/kafka.md)
- [Safe database migrations](../databases/safe-database-migrations.md)

### Compare with
- [Outbox pattern](../architecture/outbox-pattern.md) — reliable publishing; still requires idempotent consumers.

## Notes / Inbox (optional)
- N/A
