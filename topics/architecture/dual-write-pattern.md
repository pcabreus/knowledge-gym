---
id: dual-write-pattern
title: "Dual-Write Pattern"
type: pattern
status: learning
importance: 40
difficulty: medium
tags: [architecture, messaging]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Dual-Write Pattern

## TL;DR (BLUF)
- Dual-write updates two systems in a single logical operation.
- Use it only when you can tolerate occasional inconsistencies.
- Trade-off: risk of partial failure and data divergence.

## Definition
**What it is:** Writing to two different systems (e.g., DB and queue) in separate steps.
**Key terms:** dual-write, inconsistency, retries.

## Why it matters
- It’s common but risky without safeguards.
- Failures between writes cause divergence.

## Scope & Non-goals
**In scope:** dual-write risks and mitigations.
**Out of scope / NOT solved by this:** exactly-once guarantees.

## Mental model / Intuition
- Two writes = two chances to fail.

## Decision rules (When to use / When not to use)
### Use it when
- You can reconcile inconsistencies later.
### Avoid it when
- You need strong consistency across systems.

## How I would use it (practical)
- **Context:** DB write plus event publish.
- **Steps:** write DB → publish event → handle failures with retries.
- **What success looks like:** low divergence and recovery playbooks.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** simple to implement.
- **Cons / Risks:** inconsistent state on partial failure.
### Alternatives
- **Outbox pattern:** safer event publication.
- **How to choose:** prefer outbox when correctness matters.

## Failure modes & Pitfalls
- Event published without DB commit.
- DB commit without event published.

## Observability (How to detect issues)
- **Metrics:** mismatch rates, retry counts.
- **Logs:** publish failures.
- **Alerts:** mismatch spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Build reconciliation process
  - [ ] Add retries with backoff

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Assuming dual-write is “good enough.”
  - **Why it’s bad:** hidden data divergence.
  - **Better approach:** use an outbox.

## Interview readiness
### “Explain it like I’m teaching”
- Dual-write updates two systems separately. It’s simple but risky because partial failures create inconsistent state, so you usually prefer an outbox.

### Trap questions (with answers)
1) **Q:** Does dual-write guarantee consistency?
   - **A:** no; failures can create divergence.
2) **Q:** Can retries fix all issues?
   - **A:** no; retries can still fail or duplicate.
3) **Q:** Is dual-write acceptable for critical data?
   - **A:** usually not without reconciliation.

### Quick self-check (5 items)
- [ ] I can define dual-write.
- [ ] I can name a trade-off.
- [ ] I can describe a failure mode.
- [ ] I can explain a mitigation.
- [ ] I can compare with outbox.

## Links (NO duplication)
### Prerequisites
- [Event-driven basics](event-driven-basics.md)

### Related topics
- [Outbox pattern](outbox-pattern.md)

### Compare with
- [Outbox pattern](outbox-pattern.md) — safer event publication.
