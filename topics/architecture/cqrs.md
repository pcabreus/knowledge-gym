---
id: cqrs
title: "CQRS (Command Query Responsibility Segregation)"
type: pattern
status: learning
importance: 55
difficulty: medium
tags: [architecture, data, scalability]
canonical: true
aliases: []
created_at: 2026-01-21
updated_at: 2026-01-21
---

# CQRS (Command Query Responsibility Segregation)

## TL;DR (BLUF)
- CQRS separates write operations (commands) from read operations (queries).
- Use it when read and write concerns diverge or need independent scaling.
- Trade-off: eventual consistency and additional infrastructure.

## Definition
**What it is:** A pattern that uses distinct models for commands (write side) and queries (read side), often with projections to build optimized read views.  
**Key terms:** command model, query model, read model, write model, projection, read-side lag.

## Why it matters
- It allows different optimization strategies for reads and writes.
- It can simplify complex write-side business rules while keeping queries fast.

## Scope & Non-goals
**In scope:** separating models and data flows for reads and writes.  
**Out of scope / NOT solved by this:** guaranteed consistency between read and write models.

## Mental model / Intuition
- Commands change state; queries ask questions.
- The read model is a projection of write-side facts.

## Decision rules (When to use / When not to use)
### Use it when
- Read and write workloads have very different needs.
- The domain has complex write-side invariants.
- You can accept eventual consistency between models.
### Avoid it when
- A single model is sufficient and simpler.
- You need strict read-after-write consistency everywhere.
- You cannot maintain projection infrastructure.

## How I would use it (practical)
- **Context:** An order system with heavy read traffic and complex validation rules.
- **Steps:**
  1) Define command APIs that enforce invariants.
  2) Persist changes and emit events.
  3) Build read models via projections optimized for queries.
  4) Monitor read-side lag and rebuild projections when needed.
- **What success looks like:** fast queries, stable write rules, and predictable read lag.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** optimized reads, clearer write-side rules, independent scaling.
- **Cons / Risks:** eventual consistency, duplicate models, and more operational overhead.
### Alternatives
- **Single model CRUD:** simpler for small systems.
- **How to choose:** apply CQRS when read and write concerns diverge significantly.

## Failure modes & Pitfalls
- Stale reads causing user confusion.
- Projections falling behind or failing silently.
- Accidental dual-writes if events and state are updated separately.

## Observability (How to detect issues)
- **Metrics:** projection lag, command failure rate, read model rebuild time.
- **Logs:** command IDs, projection errors, read model version.
- **Traces:** command → event → projection update timing.
- **Alerts:** sustained lag or repeated projection failures.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Make commands idempotent
  - [ ] Version read models and projections
  - [ ] Provide backfill and rebuild mechanisms
- **Security / Compliance notes**
  - Ensure audit logs for command execution.
- **Performance notes**
  - Keep read models denormalized for query speed.
- **Operational notes**
  - Monitor and alert on read-side lag.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Applying CQRS everywhere by default.
  - **Why it’s bad:** adds complexity without clear benefits.
  - **Better approach:** use it only where read/write divergence is real.

## Interview readiness
### “Explain it like I’m teaching”
- CQRS separates how you change state from how you read it, which helps scale and optimize each side independently but introduces eventual consistency.

### Trap questions (with answers)
1) **Q:** Does CQRS require event sourcing?
   - **A:** No; it can be implemented with traditional persistence too.
2) **Q:** Must read and write models use different databases?
   - **A:** No; they can share the same database while remaining logically separate.
3) **Q:** Does CQRS guarantee better performance?
   - **A:** Not necessarily; it adds overhead and only helps when read/write needs diverge.

### Quick self-check (5 items)
- [ ] I can define CQRS precisely in 2–3 sentences.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Transactions](../databases/transactions.md)
- [Data modeling basics](../databases/data-modeling-basics.md)
- [Event-driven architecture](event-driven-basics.md)

### Related topics
- [Event sourcing](event-sourcing.md)
- [Domain-Driven Design (DDD)](domain-driven-design.md)
- [Outbox pattern](outbox-pattern.md)
- [Dual-write pattern](dual-write-pattern.md)

### Compare with
- [Event sourcing](event-sourcing.md) — CQRS separates read/write models; event sourcing stores change history.

## Notes / Inbox (optional)
- N/A
