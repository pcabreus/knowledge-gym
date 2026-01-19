---
id: data-modeling-basics
title: "Data Modeling Basics"
type: concept
status: learning
importance: 40
difficulty: easy
tags: [database, modeling]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Data Modeling Basics

## TL;DR (BLUF)
- Data modeling defines how data is structured and related.
- Use it to align storage with access patterns and correctness.
- Trade-off: more structure can reduce flexibility.

## Definition
**What it is:** The process of structuring entities, attributes, and relationships.
**Key terms:** entity, relationship, schema.

## Why it matters
- Good models reduce bugs and improve query performance.
- Bad models cause duplication and costly migrations.

## Scope & Non-goals
**In scope:** entities, relationships, and key choices.
**Out of scope / NOT solved by this:** normalization/denormalization tactics and performance tuning.

## Mental model / Intuition
- Modeling is choosing the shape of your data before storing it.

## Decision rules (When to use / When not to use)
### Use it when
- You’re designing new tables or collections.
### Avoid it when
- Data is transient and disposable (rare).

## How I would use it (practical)
- **Context:** New feature storing user preferences.
- **Steps:** identify entities → choose keys → decide normalization.
- **What success looks like:** consistent data and clear queries.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** clarity and correctness.
- **Cons / Risks:** schema changes can be costly.
### Alternatives
- **Schema-less storage:** faster iteration but riskier.
- **How to choose:** model upfront when correctness matters.

## Failure modes & Pitfalls
- Over-modeling for unproven requirements.

## Observability (How to detect issues)
- **Metrics:** migration frequency, query complexity.
- **Logs:** migration errors.
- **Alerts:** repeated schema rollbacks.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define entities and keys
  - [ ] Decide normalization level

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Designing tables around UI views only.
  - **Why it’s bad:** data model becomes fragile.
  - **Better approach:** model around business entities and access patterns.

## Interview readiness
### “Explain it like I’m teaching”
- Data modeling is deciding how to structure data so it’s consistent and easy to query. It balances correctness, performance, and flexibility.

### Trap questions (with answers)
1) **Q:** Is schema design optional in NoSQL?
   - **A:** no; you still model for access patterns.
2) **Q:** Should you model only for current queries?
   - **A:** mostly, but keep future changes in mind.
3) **Q:** Is denormalization always bad?
   - **A:** no; it’s a performance trade-off.

### Quick self-check (5 items)
- [ ] I can define data modeling.
- [ ] I can state a trade-off.
- [ ] I can name a pitfall.
- [ ] I can describe a modeling step.
- [ ] I can compare normalization vs denormalization.

## Links (NO duplication)
### Prerequisites
- [Requirements basics](../system-design/requirements-basics.md)

### Related topics
- [Normalization](normalization.md)
- [Relational model basics](relational-model-basics.md)

### Compare with
- [NoSQL access patterns](nosql-access-patterns.md) — modeling for queries vs relations.
