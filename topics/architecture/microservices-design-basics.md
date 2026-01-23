---
id: microservices-design-basics
title: "Microservices Design Basics"
type: concept
status: learning
importance: 75
difficulty: hard
tags: [architecture, microservices]
canonical: true
aliases: []
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Microservices Design Basics

## TL;DR (BLUF)
- Good microservices are defined by *domain boundaries*, not tech layers.
- Choose sync vs async deliberately and version your contracts.
- Trade-off: independence vs operational complexity.

## Definition
**What it is:** An architectural style that splits a system into independently deployable services aligned to domain boundaries.
**Key terms:** bounded context (bounded context), service boundary, contract (contract), autonomy.

## Why it matters
- Poor boundaries create a distributed monolith with high coordination costs.
- Good boundaries improve team autonomy and change velocity.

## Scope & Non-goals
**In scope:** boundary selection, communication patterns, contract evolution.
**Out of scope / NOT solved by this:** organizational design or infra platform details.

## Mental model / Intuition
- Each service owns a slice of the business and exposes a stable contract.

## Decision rules (When to use / When not to use)
### Use it when
- The domain is complex and teams need independent velocity.
- You can invest in platform tooling, observability, and reliability practices.

### Avoid it when
- The system is small or changing rapidly without clear boundaries.
- Operational maturity is low (on-call, CI/CD, observability not ready).

## How I would use it (practical)
- **Context:** Debt-collection platform with multiple channels and payment logic.
- **Steps:**
  1) Define [Bounded contexts](bounded-contexts.md).
  2) Decide sync vs async per interaction.
  3) Version APIs and events.
  4) Add SLOs, tracing, and runbooks.
- **What success looks like:** services can deploy independently without breaking consumers.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** autonomy, scalability, tech flexibility.
- **Cons / Risks:** distributed debugging, latency, data consistency issues.

### Alternatives
- **Modular monolith:** simpler ops and easier refactoring early on.
- **How to choose:** default to modular monolith unless you have strong boundary + ops maturity.

## Failure modes & Pitfalls
- Splitting by technical layers (API/DB) instead of domain.
- Excessive synchronous calls → [Cascading failure](../operations/cascading-failure.md).
- Contract changes without versioning.

## Observability (How to detect issues)
- **Metrics:** cross-service latency p95/p99, error rate per dependency, retry rate.
- **Logs:** correlation IDs across services.
- **Traces:** long critical paths across multiple services.
- **Alerts:** dependency-induced incident spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Domain boundaries defined
  - [ ] Contract versioning policy
  - [ ] Sync vs async decision documented
  - [ ] SLOs per service

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** “Microservices for every CRUD.”
  - **Why it’s bad:** increases overhead without clear benefits.
  - **Better approach:** start modular and split when boundaries are proven.

## Interview readiness
### “Explain it like I’m teaching”
- Microservices split a system by domain boundaries so teams can deploy independently, but the trade-off is operational complexity and distributed correctness.

### Trap questions (with answers)
1) **Q:** Are microservices always more scalable?
   - **A:** Not automatically; scaling depends on actual bottlenecks and architecture quality.
2) **Q:** Should every service have its own database?
   - **A:** Often yes for autonomy, but data sharing requires careful integration patterns.
3) **Q:** Is synchronous communication always bad?
   - **A:** No; it’s appropriate for tight user-facing workflows when latency and consistency matter.

### Quick self-check (5 items)
- [ ] I can explain when microservices are a bad idea.
- [ ] I can define good boundaries.
- [ ] I can discuss sync vs async trade-offs.
- [ ] I can explain contract versioning.
- [ ] I can name a failure mode.

## Links (NO duplication)
### Prerequisites
- [Bounded contexts](bounded-contexts.md)
- [Distributed systems basics](../system-design/distributed-systems-basics.md)

### Related topics
- [Sync vs async communication](../system-design/sync-vs-async-communication.md)
- [Versioning APIs and events](../system-design/versioning-apis-and-events.md)
- [Event-driven architecture](event-driven-basics.md)

### Compare with
- [Hexagonal architecture](hexagonal-architecture.md) — microservices define deployment boundaries; hexagonal defines code boundaries inside a service.

## Notes / Inbox (optional)
- N/A
