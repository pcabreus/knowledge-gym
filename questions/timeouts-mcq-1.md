---
id: timeouts-mcq-1
topic: ../topics/operations/timeouts.md
kind: mcq
difficulty: medium
timebox_minutes: 4
created_at: 2026-01-22
updated_at: 2026-01-22
studied: true
studied_at: 2026-01-22
score: 10/10
---

# Timeouts and budgets

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
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

## Evaluation (latest)
### Performance overview
- Score: 10/10 (100%)
- Proficiency level: Expert / Master

### Summary
- Flawless understanding of deadlines, propagation, and cancellation across services.

### Key strengths
- Correctly distinguishes timeout (duration) vs deadline (absolute time) and the role of context propagation.
- Emphasizes cancellation propagation to avoid orphaned work and resource exhaustion.
- Handles fan-out scenarios with a shared wall-clock deadline.
- Recognizes the need for safety margins and correct client/server timeout ordering.

### Areas for improvement
- None identified.

### Next level challenge
- Study request hedging and gradient timeouts.
