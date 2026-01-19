---
id: document-databases
title: "Document Databases (Overview)"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [database, nosql]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Document Databases (Overview)

## TL;DR (BLUF)
- Document databases store data as JSON-like documents with flexible schema.
- Use them for hierarchical data and rapid schema evolution.
- Trade-off: limited joins and consistency trade-offs.

## Definition
**What it is:** A database model where each record is a document with nested fields.
**Key terms:** document, embedding, referencing, schema flexibility.

## Why it matters
- It enables faster iteration for evolving product data.
- Poor modeling can cause duplication and inconsistent data.

## Scope & Non-goals
**In scope:** document model concepts and trade-offs.
**Out of scope / NOT solved by this:** vendor-specific behavior (MongoDB details).

## Mental model / Intuition
- A document is a self-contained object representing a real-world entity.

## Decision rules (When to use / When not to use)
### Use it when
- Data is naturally hierarchical and read together.
- You need flexible schemas with quick iteration.
### Avoid it when
- You need strong relational constraints and joins.

## How I would use it (practical)
- **Context:** Product catalog with variable attributes.
- **Steps:** decide embed vs reference → index critical fields → enforce schema in app.
- **What success looks like:** low query latency with minimal duplication.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** flexible schema, natural nested modeling.
- **Cons / Risks:** duplication, weaker constraints.
### Alternatives
- **PostgreSQL + JSONB:** flexible fields with relational constraints.
- **How to choose:** use documents when nested reads dominate and schema changes often.

## Failure modes & Pitfalls
- Over-embedding leading to large documents.
- Excessive references causing multiple round trips.

## Observability (How to detect issues)
- **Metrics:** document size growth, query latency.
- **Logs:** slow query logs.
- **Alerts:** rising latency on document queries.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define embed vs reference rules
  - [ ] Index high-traffic fields

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Embedding everything without size limits.
  - **Why it’s bad:** large docs and slow writes.
  - **Better approach:** embed only what is read together.

## Interview readiness
### “Explain it like I’m teaching”
- Document databases store JSON-like documents, which makes them great for hierarchical data and schema changes. The cost is fewer constraints and harder joins.

### Trap questions (with answers)
1) **Q:** Do document databases remove the need for schema design?
   - **A:** no; you still design for access patterns.
2) **Q:** Are document databases always faster?
   - **A:** no; performance depends on indexing and modeling.
3) **Q:** Can you enforce relational constraints easily?
   - **A:** not like in relational databases.

### Quick self-check (5 items)
- [ ] I can define a document database.
- [ ] I can state when to use it.
- [ ] I can explain a trade-off.
- [ ] I can give a modeling example.
- [ ] I can name a pitfall.

## Links (NO duplication)
### Prerequisites
- [NoSQL access patterns](nosql-access-patterns.md)

### Related topics
- [MongoDB documents](mongodb-documents.md)

### Compare with
- [PostgreSQL](postgresql.md) — relational constraints vs document flexibility.
