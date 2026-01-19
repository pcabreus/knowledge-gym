---
id: dynamodb-gsi-lsi
title: "DynamoDB GSI/LSI"
type: concept
status: learning
importance: 60
difficulty: hard
tags: [database, dynamodb, nosql]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# DynamoDB GSI/LSI

## TL;DR (BLUF)
- GSIs and LSIs provide secondary access patterns in DynamoDB.
- Use them when you need additional query patterns beyond PK/SK.
- Trade-off: extra write cost and complexity.

## Definition
**What it is:** Secondary indexes in DynamoDB: Global Secondary Index (GSI) and Local Secondary Index (LSI).
**Key terms:** GSI, LSI, projection, write amplification.

## Why it matters
- They enable alternate query paths without full table scans.
- Overuse increases write costs and latency.

## Scope & Non-goals
**In scope:** secondary index purpose and trade-offs.
**Out of scope / NOT solved by this:** ad-hoc analytics or joins.

## Mental model / Intuition
- Think of GSIs/LSIs as extra “views” of the same data keyed differently.

## Decision rules (When to use / When not to use)
### Use it when
- You need secondary query patterns not served by PK/SK.
### Avoid it when
- You can model with a single-table design without extra indexes.

## How I would use it (practical)
- **Context:** Query items by status in addition to user.
- **Steps:** add GSI on status → project needed attributes → monitor write cost.
- **What success looks like:** secondary queries under SLA without heavy scans.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** enable new access patterns.
- **Cons / Risks:** write amplification and extra cost.
### Alternatives
- **Table redesign:** adjust keys to avoid extra indexes.
- **How to choose:** add only when access patterns are real and frequent.

## Failure modes & Pitfalls
- GSI throttling under heavy writes.
- Projecting too many attributes increases cost.

## Observability (How to detect issues)
- **Metrics:** GSI write throttles, index size, query latency.
- **Logs:** throttling errors.
- **Alerts:** increasing GSI write errors.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Confirm access pattern necessity
  - [ ] Project minimal attributes
  - [ ] Monitor write cost

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Adding GSIs for hypothetical queries.
  - **Why it’s bad:** extra cost without benefit.
  - **Better approach:** add indexes only for proven queries.

## Interview readiness
### “Explain it like I’m teaching”
- GSIs and LSIs are DynamoDB’s secondary indexes. They let you query by alternative keys but add write cost and complexity.

### Trap questions (with answers)
1) **Q:** Are GSIs free to add?
   - **A:** no; they increase write cost and can throttle.
2) **Q:** Can LSIs have different partition keys?
   - **A:** no; LSIs share the base table partition key.
3) **Q:** Do GSIs guarantee strong consistency?
   - **A:** no; GSI queries are eventually consistent.

### Quick self-check (5 items)
- [ ] I can define GSI and LSI.
- [ ] I can state when to use them.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain GSI consistency.

## Links (NO duplication)
### Prerequisites
- [DynamoDB keys](dynamodb-keys.md)

### Related topics
- [DynamoDB](dynamodb.md)

### Compare with
- [Index](index.md) — relational indexes vs DynamoDB secondary indexes.
