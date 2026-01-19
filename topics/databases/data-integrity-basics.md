---
id: data-integrity-basics
title: "Data Integrity Basics"
type: concept
status: learning
importance: 45
difficulty: easy
tags: [database, modeling]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Data Integrity Basics

## TL;DR (BLUF)
- Data integrity ensures data is correct, consistent, and trustworthy.
- Use constraints and validation to enforce invariants.
- Trade-off: stricter checks can slow writes.

## Definition
**What it is:** The correctness and consistency of stored data across operations.
**Key terms:** constraints, invariants, validation.

## Why it matters
- Poor integrity leads to incorrect decisions and bugs.
- Integrity violations compound over time.

## Scope & Non-goals
**In scope:** core integrity concepts and enforcement layers.
**Out of scope / NOT solved by this:** cross-system integrity guarantees.

## Mental model / Intuition
- Integrity is a safety net that prevents impossible states.

## Decision rules (When to use / When not to use)
### Use it when
- Data correctness is critical to business logic.
### Avoid it when
- Data is ephemeral and correctness is less important (rare).

## How I would use it (practical)
- **Context:** User email uniqueness.
- **Steps:** add unique constraint → validate in app → handle conflict.
- **What success looks like:** no duplicates under concurrency.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** strong correctness.
- **Cons / Risks:** write overhead.
### Alternatives
- **App-only validation:** more flexible but weaker guarantees.
- **How to choose:** enforce core invariants in the DB.

## Failure modes & Pitfalls
- Relying only on app checks under race conditions.

## Observability (How to detect issues)
- **Metrics:** constraint violations.
- **Logs:** integrity errors.
- **Alerts:** spikes in violations.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Identify invariants
  - [ ] Add constraints

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Skipping constraints to move faster.
  - **Why it’s bad:** long-term data corruption.
  - **Better approach:** add minimal core constraints.

## Interview readiness
### “Explain it like I’m teaching”
- Data integrity means the database never stores invalid states. Constraints and validations enforce it; the trade-off is extra write checks.

### Trap questions (with answers)
1) **Q:** Can you rely only on app validation?
   - **A:** not safely under concurrency.
2) **Q:** Do constraints eliminate all errors?
   - **A:** no; business rules still require app logic.
3) **Q:** Is integrity only about uniqueness?
   - **A:** no; it includes correctness of all invariants.

### Quick self-check (5 items)
- [ ] I can define data integrity.
- [ ] I can name enforcement layers.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain a monitoring signal.

## Links (NO duplication)
### Prerequisites
- [ACID properties](acid-properties.md)

### Related topics
- [Constraints vs app-level enforcement](constraints-vs-app.md)

### Compare with
- [API validation](../system-design/api-validation.md)
