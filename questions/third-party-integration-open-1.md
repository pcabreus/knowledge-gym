---
id: third-party-integration-open-1
topic: ../topics/operations/third-party-integration-resilience.md
kind: open
difficulty: hard
timebox_minutes: 10
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Resilient third-party integration

## Prompt
Design a resilient integration with a voice provider. Include timeouts, retries, circuit breaker, and idempotency. Describe how you classify transient vs permanent errors.

## Answer format (what “good” looks like)
- BLUF strategy
- Error taxonomy and policy
- Observability signals

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
- Retrying permanent errors.
- No idempotency for provider timeouts.
