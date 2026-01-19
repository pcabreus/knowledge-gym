---
id: pacelc
title: "PACELC"
type: concept
status: learning
importance: 40
difficulty: medium
tags: [system-design, distributed]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# PACELC

## TL;DR (BLUF)
- PACELC extends CAP with latency trade-offs during normal operation.
- Use it to reason about consistency vs latency even without partitions.
- Trade-off: stronger consistency often increases latency.

## Definition
**What it is:** A model stating: if Partition (P), choose Availability (A) or Consistency (C); Else (E), choose Latency (L) or Consistency (C).
**Key terms:** latency, consistency, partition.

## Why it matters
- It explains why systems choose between latency and consistency even when healthy.

## Scope & Non-goals
**In scope:** conceptual trade-offs for distributed systems.
**Out of scope / NOT solved by this:** formal proofs.

## Mental model / Intuition
- Even without partitions, you still trade latency for consistency.

## Decision rules (When to use / When not to use)
### Use it when
- Evaluating distributed DB consistency choices.
### Avoid it when
- You’re on a single-node DB.

## How I would use it (practical)
- **Context:** Choosing between read replicas or strong consistency.
- **Steps:** identify latency goals → choose consistency level → document behavior.
- **What success looks like:** predictable latency and correctness.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** clearer trade-off framing.
- **Cons / Risks:** still an abstraction.
### Alternatives
- **CAP theorem:** partition-only view.
- **How to choose:** use PACELC for normal-operation trade-offs.

## Failure modes & Pitfalls
- Ignoring latency costs of strong consistency.

## Observability (How to detect issues)
- **Metrics:** latency p95, stale read reports.
- **Logs:** consistency errors.
- **Alerts:** latency spikes under strong consistency.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Document latency vs consistency expectations

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Assuming CAP explains everything.
  - **Why it’s bad:** ignores latency trade-offs in normal conditions.
  - **Better approach:** use PACELC to include latency.

## Interview readiness
### “Explain it like I’m teaching”
- PACELC says: under partitions, you choose availability or consistency; otherwise, you choose latency or consistency. It extends CAP with normal-operation trade-offs.

### Trap questions (with answers)
1) **Q:** Does PACELC replace CAP?
   - **A:** no; it extends it.
2) **Q:** Is latency only a concern during partitions?
   - **A:** no; it matters always.
3) **Q:** Does PACELC guarantee correctness?
   - **A:** no; it’s a model.

### Quick self-check (5 items)
- [ ] I can define PACELC.
- [ ] I can explain P/EL/C choices.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can relate it to CAP.

## Links (NO duplication)
### Prerequisites
- [CAP theorem (practical)](cap-theorem.md)

### Related topics
- [Consistency models](../databases/consistency-models.md)

### Compare with
- [CAP theorem (practical)](cap-theorem.md) — partitions vs latency trade-offs.
