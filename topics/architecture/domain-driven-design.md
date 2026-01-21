---
id: domain-driven-design
title: "Domain-Driven Design (DDD)"
type: concept
status: learning
importance: 60
difficulty: hard
tags: [architecture, modeling, design]
canonical: true
aliases: []
created_at: 2026-01-21
updated_at: 2026-01-21
---

# Domain-Driven Design (DDD)

## TL;DR (BLUF)
- DDD aligns software with the business domain by modeling core concepts explicitly.
- Use it for complex domains with long-lived systems and many rules.
- Trade-off: higher upfront modeling effort and ongoing discipline.

## Definition
**What it is:** A design approach that centers the domain model and a shared language between engineers and domain experts, ensuring code mirrors business concepts.  
**Key terms:** domain model, ubiquitous language, bounded context, aggregate, entity, value object, repository.

## Why it matters
- It reduces semantic drift between business needs and the codebase.
- It improves modularity by carving the domain into clear boundaries.

## Scope & Non-goals
**In scope:** modeling domain concepts, defining boundaries, and enforcing invariants.  
**Out of scope / NOT solved by this:** team organization, microservice adoption, or infrastructure choices.

## Mental model / Intuition
- Think of the model as the “source of truth” for how the business works.
- The ubiquitous language becomes the contract between people and code.

## Decision rules (When to use / When not to use)
### Use it when
- The domain is complex with many rules and edge cases.
- The system is long-lived and needs to evolve safely.
- Multiple teams need alignment on core business concepts.
### Avoid it when
- The domain is simple CRUD with little business logic.
- Speed-to-market is the only priority and the system is short-lived.
- There is no access to domain experts or shared language.

## How I would use it (practical)
- **Context:** A payments platform with complex rules and compliance requirements.
- **Steps:**
  1) Run domain discovery with experts to define the ubiquitous language.
  2) Identify bounded contexts and define their APIs.
  3) Model aggregates and invariants within each context.
  4) Keep integrations explicit via events or APIs.
- **What success looks like:** consistent terminology, fewer logic bugs, and clear ownership of domain rules.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** clearer business alignment, better modularity, safer evolution.
- **Cons / Risks:** higher upfront effort, modeling paralysis, and drift if discipline fades.
### Alternatives
- **Transaction script / CRUD-first design:** faster for simple domains, but less expressive.
- **[Data modeling basics](../databases/data-modeling-basics.md):** focuses on schemas rather than behavior.
- **How to choose:** apply DDD when domain complexity and longevity justify the investment.

## Failure modes & Pitfalls
- Ubiquitous language not adopted, leading to fragmented terms.
- Overly large bounded contexts creating a “distributed monolith.”
- Anemic domain models where behavior is pushed into services.

## Observability (How to detect issues)
- **Metrics:** frequency of invariant violations, change lead time for domain rules.
- **Logs:** domain events or audit logs indicating rule enforcement failures.
- **Traces:** cross-context flows showing excessive coupling.
- **Alerts:** spikes in validation errors or rule override rates.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define bounded contexts and ownership
  - [ ] Document ubiquitous language
  - [ ] Capture invariants in aggregates
- **Security / Compliance notes**
  - Ensure domain rules enforce compliance constraints.
- **Performance notes**
  - Keep aggregate boundaries small to reduce contention.
- **Operational notes**
  - Review model changes with domain experts regularly.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** One global domain model shared across all contexts.
  - **Why it’s bad:** increases coupling and slows evolution.
  - **Better approach:** bounded contexts with explicit integration contracts.

## Interview readiness
### “Explain it like I’m teaching”
- DDD is about building software that mirrors the business domain, using a shared language and clear boundaries so rules live in the right place.

### Trap questions (with answers)
1) **Q:** Is DDD only for microservices?
   - **A:** No; it’s a design approach that applies to any architecture.
2) **Q:** Does DDD mean modeling everything in the business?
   - **A:** No; focus on the core domain and avoid over-modeling.
3) **Q:** Are bounded contexts just team boundaries?
   - **A:** Not necessarily; they’re conceptual boundaries that may influence team structure.

### Quick self-check (5 items)
- [ ] I can define DDD precisely in 2–3 sentences.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Data modeling basics](../databases/data-modeling-basics.md)
- [Requirements basics](../system-design/requirements-basics.md)
- (TODO) Bounded contexts

### Related topics
- [Event-driven architecture](event-driven-basics.md)
- [Event sourcing](event-sourcing.md)
- [CQRS](cqrs.md)

### Compare with
- [Data modeling basics](../databases/data-modeling-basics.md) — DDD models behavior and invariants, not just schema.

## Notes / Inbox (optional)
- N/A
