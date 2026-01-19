---
id: consistency-models
title: "Consistency Models"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [database, distributed]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Consistency Models

## TL;DR (BLUF)
- Consistency models define how fresh reads are relative to writes.
- Use them to reason about correctness in distributed systems.
- Trade-off: stronger consistency can reduce availability or latency.

## Definition
**What it is:** Guarantees about read visibility across replicas and time.
**Key terms:** strong consistency, eventual consistency, read-your-writes.

## Why it matters
- It determines whether users can see stale data.
- It influences design choices and failure handling.

## Scope & Non-goals
**In scope:** high-level consistency guarantees and trade-offs.
**Out of scope / NOT solved by this:** formal proofs or consensus algorithms.

## Mental model / Intuition
- Think of consistency as how “in sync” copies of data are.

## Decision rules (When to use / When not to use)
### Use it when
- You need correctness guarantees for user flows.
### Avoid it when
- Slight staleness is acceptable and availability matters more.

## How I would use it (practical)
- **Context:** User updates profile then reads it.
- **Steps:** require strong reads for critical flows → allow eventual for feed data.
- **What success looks like:** correct UX with acceptable latency.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** stronger correctness with strong consistency.
- **Cons / Risks:** higher latency or reduced availability.
### Alternatives
- **Client-side caching:** for performance with validation.
- **How to choose:** use the weakest consistency that meets correctness needs.

## Failure modes & Pitfalls
- Assuming eventual consistency is always fresh.
- Mixing consistency levels without clear rules.

## Observability (How to detect issues)
- **Metrics:** stale read reports, latency p95.
- **Logs:** consistency-related errors.
- **Alerts:** spikes in stale read complaints.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Identify correctness-critical reads
  - [ ] Document consistency expectations

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using strong consistency everywhere.
  - **Why it’s bad:** higher latency and cost.
  - **Better approach:** use strong reads only where required.

## Interview readiness
### “Explain it like I’m teaching”
- Consistency models describe how quickly reads reflect writes. Strong consistency gives fresh data but can be slower; eventual consistency is faster but can be stale.

### Trap questions (with answers)
1) **Q:** Does eventual consistency mean “always stale”?
   - **A:** no; it just allows staleness in some cases.
2) **Q:** Is strong consistency always available?
   - **A:** no; it can reduce availability during partitions.
3) **Q:** Can you mix consistency levels?
   - **A:** yes, but you must be explicit about expectations.

### Quick self-check (5 items)
- [ ] I can define consistency models.
- [ ] I can explain a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can name a decision rule.
- [ ] I can relate it to CAP.

## Links (NO duplication)
### Prerequisites
- [CAP theorem (practical)](../system-design/cap-theorem.md)

### Related topics
- [DynamoDB consistency](dynamodb-consistency.md)

### Compare with
- [Isolation levels (general)](isolation-levels.md)
