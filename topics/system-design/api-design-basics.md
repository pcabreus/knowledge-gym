---
id: api-design-basics
title: "API Design Basics"
type: concept
status: learning
importance: 45
difficulty: easy
tags: [system-design, api]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# API Design Basics

## TL;DR (BLUF)
- API design defines resources, methods, and contracts.
- Use it to ensure clarity, consistency, and evolvability.
- Trade-off: stricter contracts can slow iteration.

## Definition
**What it is:** The process of defining API endpoints, payloads, and behavior.
**Key terms:** resource, method, contract, versioning.

## Why it matters
- It reduces integration bugs and ambiguity.
- Poor design leads to breaking changes and confusion.

## Scope & Non-goals
**In scope:** API structure and contracts.
**Out of scope / NOT solved by this:** implementation details.

## Mental model / Intuition
- An API is a contract; be explicit and consistent.

## Decision rules (When to use / When not to use)
### Use it when
- You expose APIs to clients or other teams.
### Avoid it when
- You only need internal prototypes with no consumers.

## How I would use it (practical)
- **Context:** Public REST API.
- **Steps:** define resources → choose methods → document responses.
- **What success looks like:** stable integrations and low breaking changes.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** clarity and consistency.
- **Cons / Risks:** slower changes.
### Alternatives
- **Loose contracts:** faster but risky.
- **How to choose:** use strict design for long-lived APIs.

## Failure modes & Pitfalls
- Overloading endpoints with multiple behaviors.

## Observability (How to detect issues)
- **Metrics:** error rate, client integration failures.
- **Logs:** contract mismatch errors.
- **Alerts:** spikes in 4xx/5xx.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define request/response schemas
  - [ ] Version breaking changes

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Changing response shape without versioning.
  - **Why it’s bad:** breaks clients.
  - **Better approach:** version or add fields compatibly.

## Interview readiness
### “Explain it like I’m teaching”
- API design is about defining stable contracts: resources, methods, and payloads. Good design prevents breaking clients and makes systems easier to integrate.

### Trap questions (with answers)
1) **Q:** Is versioning always required?
   - **A:** for breaking changes, yes.
2) **Q:** Should APIs expose database tables directly?
   - **A:** no; design around use cases.
3) **Q:** Are optional fields always safe?
   - **A:** no; they can create ambiguity.

### Quick self-check (5 items)
- [ ] I can define API design basics.
- [ ] I can state a trade-off.
- [ ] I can name a pitfall.
- [ ] I can describe a monitoring signal.
- [ ] I can explain versioning.

## Links (NO duplication)
### Prerequisites
- N/A

### Related topics
- [API validation](api-validation.md)

### Compare with
- [Constraints vs app-level enforcement](../databases/constraints-vs-app.md) — API vs DB rules.
