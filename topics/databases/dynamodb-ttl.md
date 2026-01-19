---
id: dynamodb-ttl
title: "DynamoDB TTL"
type: concept
status: learning
importance: 45
difficulty: easy
tags: [database, dynamodb]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# DynamoDB TTL

## TL;DR (BLUF)
- DynamoDB TTL is a DynamoDB implementation of [TTL (Time-to-Live)](ttl.md).
- Use it for ephemeral data and cache-like records.
- Trade-off: deletions are eventual and not immediate.

## Definition
**What it is:** A DynamoDB feature that removes items when a designated TTL attribute expires.
**Key terms:** TTL attribute, expiration, eventual deletion.

## Why it matters
- It automates cleanup and reduces storage cost.
- It’s not precise; deletion timing is best-effort.

## Scope & Non-goals
**In scope:** automatic expiry of items.
**Out of scope / NOT solved by this:** real-time deletion guarantees.

## Mental model / Intuition
- Think of TTL as a background janitor, not a precise timer.

## Decision rules (When to use / When not to use)
### Use it when
- Data has a natural expiration (sessions, temporary tokens).
### Avoid it when
- You need exact deletion timing.

## How I would use it (practical)
- **Context:** Session records with expiration.
- **Steps:** add TTL attribute → enable TTL → monitor deletion rate.
- **What success looks like:** expired data disappears without manual cleanup.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** automated cleanup.
- **Cons / Risks:** delayed deletion; possible stale reads.
### Alternatives
- **Scheduled jobs:** precise cleanup.
- **How to choose:** use TTL for best-effort cleanup; use jobs for strict timing.

## Failure modes & Pitfalls
- Relying on TTL for strict security enforcement.

## Observability (How to detect issues)
- **Metrics:** TTL deletion rate, storage growth.
- **Logs:** audit expired items in app logic.
- **Alerts:** storage not decreasing as expected.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Set TTL attribute in epoch seconds
  - [ ] Don’t rely on TTL for immediate deletion

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using TTL as a security control.
  - **Why it’s bad:** deletion is delayed.
  - **Better approach:** enforce expiration in application logic.

## Interview readiness
### “Explain it like I’m teaching”
- DynamoDB TTL removes items after they expire, but it’s best-effort and not immediate. It’s great for cleanup, not for strict timing guarantees.

### Trap questions (with answers)
1) **Q:** Does TTL delete items exactly at expiration time?
   - **A:** no; deletion is eventual.
2) **Q:** Can TTL be used for real-time session invalidation?
   - **A:** no; use app-level checks.
3) **Q:** Does TTL generate Streams events?
   - **A:** yes; TTL deletions can appear in Streams.

### Quick self-check (5 items)
- [ ] I can define TTL precisely.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain deletion timing.

## Links (NO duplication)
### Prerequisites
- [DynamoDB](dynamodb.md)
- [TTL (Time-to-Live)](ttl.md)

### Related topics
- [DynamoDB Streams](dynamodb-streams.md)

### Compare with
- [Scheduled cleanup jobs](../operations/scheduled-cleanup-jobs.md)
