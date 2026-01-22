---
id: versioning-apis-and-events
title: "Versioning APIs and Events"
type: concept
status: learning
importance: 80
difficulty: medium
tags: [system-design, api, messaging]
canonical: true
aliases: []
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Versioning APIs and Events

## TL;DR (BLUF)
- Versioning protects consumers from breaking changes.
- Prefer backward-compatible changes; introduce new versions only when required.
- Event versioning must handle replay and old consumers.

## Definition
**What it is:** A strategy for evolving API contracts and event schemas without breaking consumers.
**Key terms:** backward compatibility (backward compatibility), schema evolution (schema evolution), contract (contract), deprecation.

## Why it matters
- Contract breaks cause outages across services and integrations.
- Versioning discipline enables independent deploys.

## Scope & Non-goals
**In scope:** REST/API and event schema evolution.
**Out of scope / NOT solved by this:** low-level serialization details.

## Mental model / Intuition
- Treat contracts like public interfaces: breaking changes require versioning and deprecation.

## Decision rules (When to use / When not to use)
### Use a new version when
- You must remove or rename fields or change semantics.
- You need incompatible behavior changes.

### Avoid new versions when
- You can add optional fields (backward-compatible).
- You can make behavior additive.

## How I would use it (practical)
- **Context:** Event `ContactAttempted` needs new fields.
- **Steps:**
  1) Add optional fields with defaults (backward-compatible).
  2) Version the schema if behavior changes.
  3) Deprecate old version with a clear sunset date.
- **What success looks like:** old consumers keep working and migrations are predictable.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** stability and independent deploys.
- **Cons / Risks:** version sprawl and higher maintenance.

### Alternatives
- **Consumer-driven contracts:** test compatibility proactively.
- **Schema registry + compatibility rules:** enforce evolution at CI.
- **How to choose:** default to backward-compatible changes; version only when necessary.

## Failure modes & Pitfalls
- Changing semantics without a version bump.
- Dropping fields that old consumers still need.
- Replaying old events with a new schema that can’t parse them.

## Observability (How to detect issues)
- **Metrics:** schema validation failures, consumer error rates by version.
- **Logs:** contract version in every request/event.
- **Traces:** version tags across services.
- **Alerts:** spike in validation errors or 4xx after deploy.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define compatibility rules
  - [ ] Add contract version to payload/headers
  - [ ] Deprecation timeline and communication plan

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** “We’ll just change it and ask consumers to update.”
  - **Why it’s bad:** breaks independent deploys and causes outages.
  - **Better approach:** backward-compatible evolution + deprecation.

## Interview readiness
### “Explain it like I’m teaching”
- Versioning keeps contracts stable for consumers; prefer backward-compatible additions and only introduce new versions for breaking changes.

### Trap questions (with answers)
1) **Q:** If you add a new required field, is that backward-compatible?
   - **A:** No; old consumers won’t send it.
2) **Q:** Can you change enum meanings without a version bump?
   - **A:** No; semantics change is breaking even if the schema parses.
3) **Q:** Are event versions only for producers?
   - **A:** No; consumers need to handle multiple versions during rollout and replays.

### Quick self-check (5 items)
- [ ] I can define backward compatibility.
- [ ] I can list breaking changes.
- [ ] I can design deprecation.
- [ ] I can handle event replays.
- [ ] I can name an observability signal.

## Links (NO duplication)
### Prerequisites
- [API design basics](api-design-basics.md)
- [Event-driven architecture](../architecture/event-driven-basics.md)

### Related topics
- [Event sourcing](../architecture/event-sourcing.md)
- [Message delivery semantics](../architecture/message-delivery-semantics.md)

### Compare with
- [API validation](api-validation.md) — validation enforces contracts but does not define evolution rules.

## Notes / Inbox (optional)
- N/A
