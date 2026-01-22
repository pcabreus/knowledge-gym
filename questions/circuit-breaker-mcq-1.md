---
id: circuit-breaker-mcq-1
topic: ../topics/operations/circuit-breaker.md
kind: mcq
difficulty: medium
timebox_minutes: 4
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Circuit breaker behavior

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
Your dependency returns 60% timeouts over the last minute. What is the best circuit breaker action?

## Answer format (what “good” looks like)
- Open breaker and fail fast
- Use half-open probes

## Answer key / Rubric
### Correct answer (for MCQ)
- Correct: A
- Why correct: A high timeout rate indicates sustained failure; opening the breaker protects resources and half-open probes allow recovery testing.
- Why A is wrong: N/A
- Why B is wrong: Keeping it closed causes more timeouts and resource waste.
- Why C is wrong: Retrying indefinitely amplifies load and worsens outages.
- Why D is wrong: “Always succeed by ignoring errors” is absurd.

A) Open the breaker, fail fast, and probe periodically (half-open).
B) Keep it closed and let timeouts resolve naturally.
C) Increase retries to compensate for timeouts.
D) Mark all requests as successful regardless of dependency response.

## Common mistakes
- Treating client errors as breaker signals.
- No half-open probes.

