---
id: api-validation
title: "API Validation"
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

# API Validation

## TL;DR (BLUF)
- API validation enforces input correctness at the boundary.
- Use it to prevent invalid data from entering the system.
- Trade-off: stricter validation can reduce flexibility.

## Definition
**What it is:** Rules and checks applied to API inputs before processing.
**Key terms:** schema validation, constraints, input sanitization.

## Why it matters
- It prevents bad data from propagating into storage.
- It reduces downstream bugs and inconsistencies.

## Scope & Non-goals
**In scope:** validating payloads and enforcing constraints.
**Out of scope / NOT solved by this:** database-level integrity guarantees.

## Mental model / Intuition
- Treat validation as the first guardrail.

## Decision rules (When to use / When not to use)
### Use it when
- External clients send data that must be verified.
### Avoid it when
- You can trust inputs and need maximum flexibility (rare).

## How I would use it (practical)
- **Context:** User signup endpoint.
- **Steps:** validate schema → enforce required fields → reject invalid inputs.
- **What success looks like:** low invalid data rate.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** cleaner data and fewer bugs.
- **Cons / Risks:** friction for clients.
### Alternatives
- **Database constraints:** stronger integrity at storage.
- **How to choose:** validate at API boundary and enforce core invariants in DB.

## Failure modes & Pitfalls
- Overly strict validation blocking valid changes.

## Observability (How to detect issues)
- **Metrics:** validation error rate.
- **Logs:** invalid payloads.
- **Alerts:** spikes in validation failures.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define schemas
  - [ ] Return actionable error messages

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Accepting any payload without validation.
  - **Why it’s bad:** data corruption risk.
  - **Better approach:** validate required fields and types.

## Interview readiness
### “Explain it like I’m teaching”
- API validation checks input at the boundary so bad data doesn’t enter the system. It complements database constraints and keeps data clean.

### Trap questions (with answers)
1) **Q:** Does API validation replace DB constraints?
   - **A:** no; DB constraints still protect integrity.
2) **Q:** Is validation only about types?
   - **A:** no; it includes business rules and ranges.
3) **Q:** Can you skip validation for internal APIs?
   - **A:** not always; internal bugs still happen.

### Quick self-check (5 items)
- [ ] I can define API validation.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain a monitoring signal.

## Links (NO duplication)
### Prerequisites
- [API design basics](api-design-basics.md)

### Related topics
- [Data integrity basics](../databases/data-integrity-basics.md)

### Compare with
- [Constraints vs app-level enforcement](../databases/constraints-vs-app.md) — DB vs API checks.
