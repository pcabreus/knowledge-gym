---
id: ttl
title: "TTL (Time-to-Live)"
type: concept
status: learning
importance: 45
difficulty: easy
tags: [database, caching]
canonical: true
aliases: ["time-to-live"]
created_at: 2026-01-19
updated_at: 2026-01-19
---

# TTL (Time-to-Live)

## TL;DR (BLUF)
- TTL defines when data expires and can be deleted automatically.
- Use it for cache entries or ephemeral records.
- Trade-off: expiration is often best-effort, not exact.

## Definition
**What it is:** A mechanism to expire data after a specified time period.
**Key terms:** expiration, eviction, best-effort deletion.

## Why it matters
- It prevents storage growth and stale data accumulation.
- Relying on precise TTL timing can cause correctness issues.

## Scope & Non-goals
**In scope:** TTL as a data lifecycle mechanism.
**Out of scope / NOT solved by this:** precise scheduling guarantees.

## Mental model / Intuition
- Think of TTL as a background cleanup job.

## Decision rules (When to use / When not to use)
### Use it when
- Data is ephemeral and can expire safely.
### Avoid it when
- You need strict deletion timing for security or compliance.

## How I would use it (practical)
- **Context:** Session tokens or temporary flags.
- **Steps:** set TTL on write → enforce expiration in reads → monitor cleanup.
- **What success looks like:** storage stays bounded and stale data is removed.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** automatic cleanup and bounded storage.
- **Cons / Risks:** deletion timing is approximate.
### Alternatives
- **Scheduled jobs:** precise expiration control.
- **How to choose:** use TTL for best-effort cleanup, jobs for strict timing.

## Failure modes & Pitfalls
- Using TTL as a security boundary.

## Observability (How to detect issues)
- **Metrics:** storage growth, deletion rate.
- **Logs:** expired-item access attempts.
- **Alerts:** storage not decreasing as expected.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Don’t rely on TTL for immediate deletion
  - [ ] Enforce expiry in app logic when needed

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Assuming TTL is exact.
  - **Why it’s bad:** delayed deletions can expose stale data.
  - **Better approach:** validate expiry at read time.

## Interview readiness
### “Explain it like I’m teaching”
- TTL marks data to expire automatically, which is great for cache and temporary records. But most systems delete items eventually, not exactly at the deadline.

### Trap questions (with answers)
1) **Q:** Does TTL guarantee immediate deletion?
   - **A:** no; it’s usually best-effort.
2) **Q:** Can TTL replace authentication checks?
   - **A:** no; enforce expiry in application logic.
3) **Q:** Is TTL only for caches?
   - **A:** no; it can be used for any ephemeral data.

### Quick self-check (5 items)
- [ ] I can define TTL precisely.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain best-effort deletion.

## Links (NO duplication)
### Prerequisites
- [Caching fundamentals](../system-design/caching-fundamentals.md)

### Related topics
- [Redis](redis.md)
- [DynamoDB TTL](dynamodb-ttl.md)

### Compare with
- [Scheduled cleanup jobs](../operations/scheduled-cleanup-jobs.md)
