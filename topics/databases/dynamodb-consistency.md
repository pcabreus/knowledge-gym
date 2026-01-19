---
id: dynamodb-consistency
title: "DynamoDB Consistency"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [database, dynamodb, consistency]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# DynamoDB Consistency

## TL;DR (BLUF)
- DynamoDB consistency is a DynamoDB-specific case of [Consistency models](consistency-models.md).
- Use strong reads when correctness outweighs latency/cost.
- Trade-off: strong reads are more expensive and may be slower.

## Definition
**What it is:** Read consistency options that determine staleness and cost in DynamoDB.
**Key terms:** eventual consistency, strong consistency, read capacity.

## Why it matters
- Consistency choices affect correctness and latency.
- Wrong choice can cause stale reads or higher cost.

## Scope & Non-goals
**In scope:** read consistency options and trade-offs.
**Out of scope / NOT solved by this:** cross-region global consistency.

## Mental model / Intuition
- Eventual consistency is “fast and cheap,” strong is “fresh and pricey.”

## Decision rules (When to use / When not to use)
### Use it when
- You need read-your-writes consistency for critical workflows.
### Avoid it when
- Slight staleness is acceptable and cost matters.

## How I would use it (practical)
- **Context:** User updates profile then reads it.
- **Steps:** use strong read after write → fall back to eventual for other reads.
- **What success looks like:** correct UX without excessive cost.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** correctness when needed.
- **Cons / Risks:** higher cost and latency.
### Alternatives
- **Client-side caching with validation:** reduce strong reads.
- **How to choose:** use strong reads sparingly for correctness-critical flows.

## Failure modes & Pitfalls
- Assuming eventual reads are always fresh.
- Overusing strong reads and increasing costs.

## Observability (How to detect issues)
- **Metrics:** read capacity consumption, latency p95.
- **Logs:** stale read reports.
- **Alerts:** cost spikes or increased latency.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Identify correctness-critical reads
  - [ ] Use strong reads selectively

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using strong reads everywhere.
  - **Why it’s bad:** higher cost and latency.
  - **Better approach:** use eventual reads where acceptable.

## Interview readiness
### “Explain it like I’m teaching”
- DynamoDB reads are eventually consistent by default, but you can request strong consistency when you need fresh data. It’s a cost/latency trade-off.

### Trap questions (with answers)
1) **Q:** Are GSIs strongly consistent?
   - **A:** no; GSI reads are eventually consistent.
2) **Q:** Does strong consistency apply across regions?
   - **A:** no; it’s within a region.
3) **Q:** Can you get stale reads after a write?
   - **A:** yes, with eventual consistency.

### Quick self-check (5 items)
- [ ] I can define DynamoDB consistency options.
- [ ] I can state when to use strong reads.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain GSI consistency.

## Links (NO duplication)
### Prerequisites
- [DynamoDB](dynamodb.md)
- [Consistency models](consistency-models.md)

### Related topics
- [DynamoDB GSI/LSI](dynamodb-gsi-lsi.md)

### Compare with
- [Consistency models](consistency-models.md)
