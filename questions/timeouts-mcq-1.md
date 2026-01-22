---
id: timeouts-mcq-1
topic: ../topics/operations/timeouts.md
kind: mcq
difficulty: medium
timebox_minutes: 4
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Timeouts and budgets

## Prompt
Which approach best prevents cascading failures in a service that calls two downstream dependencies?

## Answer format (what “good” looks like)
- Use time budgets and per-hop timeouts
- Mention cancellation propagation

## Answer key / Rubric
### Correct answer (for MCQ)
- Correct: B
- Why correct: Allocate a time budget across downstream calls and propagate cancellation (cancellation) so work stops when the deadline is reached.
- Why A is wrong: Using a single large timeout hides latency issues and increases resource contention.
- Why B is wrong: N/A
- Why C is wrong: Disabling timeouts causes stuck work and resource exhaustion.
- Why D is wrong: Retrying without timeouts creates uncontrolled concurrency.

A) Set a single 10s timeout for the whole request and hope it’s enough.
B) Set per-hop timeouts within an end-to-end budget and propagate cancellation.
C) Disable timeouts to avoid false failures.
D) Retry each downstream call indefinitely.

## Common mistakes
- No end-to-end budget.
- Not canceling work after timeout.
