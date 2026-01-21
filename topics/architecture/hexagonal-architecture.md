---
id: hexagonal-architecture
title: "Hexagonal Architecture"
type: pattern
status: learning
importance: 55
difficulty: medium
tags: [architecture, design, modularity]
canonical: true
aliases: []
created_at: 2026-01-21
updated_at: 2026-01-21
---

# Hexagonal Architecture

## TL;DR (BLUF)
- Hexagonal architecture keeps the application core independent from external systems using ports and adapters.
- Use it to improve testability and swap infrastructure without touching domain logic.
- Trade-off: extra abstraction layers and upfront design effort.

## Definition
**What it is:** An architectural pattern that isolates the application core behind well-defined ports (interfaces), with adapters implementing integrations (DB, [HTTP](../operations/http.md), messaging) at the edges.  
**Key terms:** ports, adapters, primary/secondary adapters, dependency inversion, application core.

## Why it matters
- It prevents infrastructure details from leaking into business logic.
- It makes testing easier by substituting adapters with fakes or mocks.

## Scope & Non-goals
**In scope:** boundaries, dependency direction, and integration design.  
**Out of scope / NOT solved by this:** domain modeling strategy or distributed consistency.

## Mental model / Intuition
- Picture a hexagon: the core is in the middle, adapters plug into the sides.
- Dependencies point inward; the core defines what it needs, adapters satisfy it.

## Decision rules (When to use / When not to use)
### Use it when
- You have multiple external integrations (DB, queues, APIs).
- You want strong testability of the application core.
- You expect infrastructure changes over time.
### Avoid it when
- The system is simple and short-lived.
- The team can’t afford the abstraction overhead.
- Tight coupling is acceptable and faster delivery is required.

## How I would use it (practical)
- **Context:** A service with a DB, a message broker, and external APIs.
- **Steps:**
  1) Define inbound ports (use cases) and outbound ports (dependencies).
  2) Keep the core free of frameworks.
  3) Implement adapters for [HTTP](../operations/http.md), DB, and messaging.
  4) Wire dependencies with DI and test with in-memory adapters.
- **What success looks like:** core logic tested without infrastructure and adapters replaceable with minimal changes.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** high testability, clear boundaries, flexibility in infrastructure changes.
- **Cons / Risks:** more boilerplate, higher cognitive load, and slower initial delivery.
### Alternatives
- **Layered architecture:** simpler for small systems but less flexible at the boundaries.
- **How to choose:** prefer hexagonal when long-term maintainability and integration churn are likely.

## Failure modes & Pitfalls
- Letting frameworks leak into the application core.
- Ports that mirror adapter details instead of domain needs.
- Over-fragmented adapters leading to complexity without benefit.

## Observability (How to detect issues)
- **Metrics:** change lead time for integrations, test coverage of the core.
- **Logs:** adapter errors separated from core validation failures.
- **Traces:** spans that clearly distinguish core logic vs adapter calls.
- **Alerts:** rising adapter error rates or increasing coupling regressions.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define ports based on use cases
  - [ ] Keep dependencies pointing inward
  - [ ] Use adapter-specific DTOs at the edges
- **Security / Compliance notes**
  - Validate inputs at adapters before entering the core.
- **Performance notes**
  - Avoid excessive mapping layers; keep adapters thin.
- **Operational notes**
  - Monitor adapter health separately from core logic.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Treating adapters as the source of truth.
  - **Why it’s bad:** it inverts dependencies and pollutes the core.
  - **Better approach:** keep business rules in the core and adapters at the edge.

## Interview readiness
### “Explain it like I’m teaching”
- Hexagonal architecture keeps the business core independent by defining ports and connecting adapters at the edges, making infrastructure replaceable and testing easier.

### Trap questions (with answers)
1) **Q:** Does hexagonal architecture force microservices?
   - **A:** No; it is a modular design approach that works within monoliths or services.
2) **Q:** Are ports just network ports?
   - **A:** No; they are interfaces that define how the core is used or what it needs.
3) **Q:** Can adapters depend on the core?
   - **A:** Yes; dependencies should point inward to the core.

### Quick self-check (5 items)
- [ ] I can define hexagonal architecture precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [SOLID principles](../quality/solid-principles.md)
- [Design patterns (overview)](../quality/design-patterns.md)
- [API design basics](../system-design/api-design-basics.md)

### Related topics
- [Domain-Driven Design (DDD)](domain-driven-design.md)
- [CQRS](cqrs.md)
- [Event-driven architecture](event-driven-basics.md)
- [Outbox pattern](outbox-pattern.md)

### Compare with
- [Event-driven architecture](event-driven-basics.md) — structure vs communication style.

## Notes / Inbox (optional)
- N/A
