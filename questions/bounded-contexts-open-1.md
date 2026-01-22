---
id: bounded-contexts-open-1
topic: ../topics/architecture/bounded-contexts.md
kind: open
difficulty: hard
timebox_minutes: 10
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Bounded contexts for debt collection

## Prompt
Given entities like `debt_case`, `contact_attempt`, `promise_to_pay`, and `payment_schedule`, propose bounded contexts and their integration contracts.

## Answer format (what “good” looks like)
- BLUF boundaries
- Context map / integration style
- 1 trade-off and 1 failure mode

## Answer key / Rubric
### Scoring rubric (for open / design / teach-back)
Score each 0–5:
1) Definition correctness
2) Decision rules (when/when not)
3) Trade-offs & alternatives
4) Example / evidence
5) Failure modes & detection
6) Clarity (BLUF)

## Common mistakes
- Splitting by technical layers instead of domain.
- No contract versioning.
