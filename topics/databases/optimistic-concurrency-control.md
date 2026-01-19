---
id: optimistic-concurrency-control
title: "Optimistic Concurrency Control"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [database, concurrency]
canonical: true
aliases: ["occ"]
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Optimistic Concurrency Control

## TL;DR (BLUF)
- Optimistic concurrency assumes conflicts are rare and checks versions on write.
- Use it for high-concurrency workloads with low conflict rates.
- Trade-off: conflicts require retries and error handling.

## Definition
**What it is:** A concurrency approach using version checks to detect conflicts at commit time.
**Key terms:** version column, compare-and-swap, retries.

## Why it matters
- It reduces locking overhead and improves concurrency.
- Poor retry handling leads to lost updates or errors.

## Scope & Non-goals
**In scope:** OCC concepts and trade-offs.
**Out of scope / NOT solved by this:** distributed transactions.

## Mental model / Intuition
- Think “check before you write; retry if changed.”

## Decision rules (When to use / When not to use)
### Use it when
- Conflicts are rare and throughput matters.
### Avoid it when
- Conflicts are common and retries would be expensive.

## How I would use it (practical)
- **Context:** Updating user profile settings.
- **Steps:** read version → update with version check → retry on conflict.
- **What success looks like:** few conflicts and minimal retries.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** high concurrency without heavy locks.
- **Cons / Risks:** retries and conflict handling complexity.
### Alternatives
- **Lock-based concurrency control:** for high-conflict workloads.
- **How to choose:** choose OCC when conflicts are low.

## Failure modes & Pitfalls
- Missing version checks leading to lost updates.

## Observability (How to detect issues)
- **Metrics:** conflict rate, retry rate.
- **Logs:** update conflicts.
- **Alerts:** spikes in retries or conflict errors.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Add version column
  - [ ] Implement retry with backoff

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Ignoring conflict errors.
  - **Why it’s bad:** silent data loss.
  - **Better approach:** surface errors and retry.

## Interview readiness
### “Explain it like I’m teaching”
- Optimistic concurrency checks a version when writing. If someone else changed the row, the write fails and you retry. It’s great when conflicts are rare.

### Trap questions (with answers)
1) **Q:** Does OCC eliminate conflicts?
   - **A:** no; it detects them and requires retries.
2) **Q:** Is OCC always better than locking?
   - **A:** no; for high-conflict workloads, locking can be better.
3) **Q:** Can OCC be done without a version field?
   - **A:** not reliably; you need a conflict detector.

### Quick self-check (5 items)
- [ ] I can define OCC.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a failure mode.
- [ ] I can explain retry behavior.

## Links (NO duplication)
### Prerequisites
- [Transactions](transactions.md)

### Related topics
- [Locks](locks.md)

### Compare with
- [Lock-based concurrency control](lock-based-concurrency-control.md)
