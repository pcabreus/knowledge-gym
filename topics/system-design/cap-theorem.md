---
id: cap-theorem
title: "CAP Theorem (Practical)"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [system-design, distributed]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# CAP Theorem (Practical)

## TL;DR (BLUF)
- CAP describes trade-offs between consistency, availability, and partition tolerance.
- Use it to reason about distributed system behavior under partitions.
- Trade-off: you can’t have full consistency and availability during partitions.

## Definition
**What it is:** A theorem describing trade-offs among consistency, availability, and partition tolerance.
**Key terms:** consistency, availability, partition tolerance.

## Why it matters
- It frames design decisions under network partitions.
- Misunderstanding leads to incorrect reliability expectations.

## Scope & Non-goals
**In scope:** practical CAP trade-offs.
**Out of scope / NOT solved by this:** formal proofs or PACELC nuances.

## Mental model / Intuition
- When the network splits, you must choose between fresh data and staying available.

## Decision rules (When to use / When not to use)
### Use it when
- Designing distributed data systems.
### Avoid it when
- You’re only running a single-node database.

## How I would use it (practical)
- **Context:** Multi-region writes.
- **Steps:** decide consistency goals → choose system settings → document behavior.
- **What success looks like:** clear expectations for stale reads or downtime.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** clearer design decisions.
- **Cons / Risks:** oversimplification if used naively.
### Alternatives
- **PACELC:** adds latency trade-offs.
- **How to choose:** use CAP for partition scenarios, then refine.

## Failure modes & Pitfalls
- Assuming CAP says “choose two” at all times.

## Observability (How to detect issues)
- **Metrics:** availability, error rate during partitions.
- **Logs:** partition-related errors.
- **Alerts:** increased timeouts or read failures.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Document consistency/availability choices
  - [ ] Test partition behavior

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Treating CAP as a daily operational choice.
  - **Why it’s bad:** it applies only under partitions.
  - **Better approach:** plan for partitions explicitly.

## Interview readiness
### “Explain it like I’m teaching”
- CAP says that during a network partition you must choose between consistent data and availability. It’s a trade-off that informs distributed system design.

### Trap questions (with answers)
1) **Q:** Can you have C and A without P?
   - **A:** not in distributed systems; partitions are inevitable.
2) **Q:** Does CAP apply to single-node DBs?
   - **A:** not really; partitions aren’t relevant.
3) **Q:** Is CAP “choose two” always?
   - **A:** only under partition conditions.

### Quick self-check (5 items)
- [ ] I can define CAP.
- [ ] I can explain when it matters.
- [ ] I can name a pitfall.
- [ ] I can describe a trade-off.
- [ ] I can connect CAP to consistency models.

## Links (NO duplication)
### Prerequisites
- [Distributed systems basics](distributed-systems-basics.md)

### Related topics
- [Consistency models](../databases/consistency-models.md)

### Compare with
- [PACELC](pacelc.md)
