---
id: idempotency-open-1
topic: ../topics/operations/idempotency.md
kind: open
difficulty: medium
timebox_minutes: 7
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Idempotency for payment requests

## Prompt
Design an idempotency strategy for a `PaymentRequested` event processed by a consumer that writes to Postgres and calls a payment provider. Explain keys, dedup storage, and failure handling.

## Answer format (what “good” looks like)
- BLUF definition
- Key choice and dedup storage
- Handling retries and third-party timeouts
- 1 failure mode + detection signal

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
- Using unstable keys or timestamps.
- Forgetting idempotency for the external API call.
