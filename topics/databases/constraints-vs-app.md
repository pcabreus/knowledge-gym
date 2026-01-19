---
id: constraints-vs-app
title: "Constraints vs App-Level Enforcement"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [database, modeling]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Constraints vs App-Level Enforcement

## TL;DR (BLUF)
- DB constraints enforce rules at the storage layer; app logic enforces them in code.
- Use constraints for core invariants and app logic for complex or cross-system rules.
- Trade-off: constraints protect integrity but can add write complexity.

## Definition
**What it is:** A comparison between database constraints (FK, unique, check) and application-level validation.
**Key terms:** constraints, invariants, validation.

## Why it matters
- Choosing the wrong layer leads to data inconsistency or reduced flexibility.
- Constraints are the last line of defense for core invariants.

## Scope & Non-goals
**In scope:** deciding what belongs in the DB vs application.
**Out of scope / NOT solved by this:** cross-service guarantees.

## Mental model / Intuition
- Constraints are guardrails at the data store; app logic is business rules.

## Decision rules (When to use / When not to use)
### Use it when
- The rule is fundamental and must always hold.
- You want guarantees even if other systems write data.
### Avoid it when
- The rule requires external context or cross-system checks.

## How I would use it (practical)
- **Context:** Unique email per user.
- **Steps:** enforce unique constraint → handle conflicts in app.
- **What success looks like:** no duplicates even under race conditions.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** strong data integrity.
- **Cons / Risks:** stricter writes, migrations may be harder.
### Alternatives
- **App-only validation:** flexible but weaker guarantees.
- **How to choose:** use DB constraints for core invariants, app for contextual rules.

## Failure modes & Pitfalls
- Relying only on app checks and getting race-condition duplicates.

## Observability (How to detect issues)
- **Metrics:** constraint violation rate.
- **Logs:** constraint errors.
- **Alerts:** spikes in constraint violations.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Identify invariants
  - [ ] Add constraints for core invariants
  - [ ] Handle constraint errors in app

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Skipping constraints to move faster.
  - **Why it’s bad:** data corruption over time.
  - **Better approach:** enforce minimal core constraints.

## Interview readiness
### “Explain it like I’m teaching”
- Constraints enforce core data rules at the database layer, while app logic handles business rules. Use constraints for invariants you never want violated.

### Trap questions (with answers)
1) **Q:** Can app-level checks replace DB constraints?
   - **A:** not reliably; race conditions can bypass them.
2) **Q:** Are constraints always worth it?
   - **A:** for core invariants, yes; for complex rules, app logic fits better.
3) **Q:** Do constraints slow every write?
   - **A:** they add overhead, but usually worth it for integrity.

### Quick self-check (5 items)
- [ ] I can state when to use constraints.
- [ ] I can describe a race-condition example.
- [ ] I can name a trade-off.
- [ ] I can explain app-level enforcement use cases.
- [ ] I can explain a monitoring signal.

## Links (NO duplication)
### Prerequisites
- [Data integrity basics](data-integrity-basics.md)

### Related topics
- [Transactions](transactions.md)

### Compare with
- [API validation](../system-design/api-validation.md)
