---
id: retries-backoff-mcq-1
topic: ../topics/operations/retries-and-backoff.md
kind: mcq
difficulty: medium
timebox_minutes: 4
created_at: 2026-01-22
updated_at: 2026-01-22
studied: true
studied_at: 2026-01-22
score: 10/10
---

# Retries with backoff and jitter

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
You are calling a third-party API with a 1s end-to-end latency budget. Which retry strategy is best?

## Answer format (what “good” looks like)
- Identify bounded retries with exponential backoff and jitter
- Respect time budget and idempotency

## Answer key / Rubric
### Correct answer (for MCQ)
- Correct: C
- Why correct: Bounded retries with exponential backoff and jitter (jitter) reduce retry storms and must fit within the total time budget, assuming the operation is idempotent.
- Why A is wrong: Infinite retries can cause outages and violate the latency budget.
- Why B is wrong: Constant retry intervals without jitter cause synchronized bursts and can exceed the budget.
- Why C is wrong: N/A
- Why D is wrong: Skipping retries ignores transient errors and reduces reliability without justification.

A) Retry until success with no delay.
B) Retry 5 times every 200ms with no jitter.
C) Retry 2–3 times with exponential backoff + jitter within the 1s budget.
D) Never retry and always fail fast.

## Common mistakes
- Ignoring time budgets.
- Retrying non-idempotent operations.

## Evaluation (latest)
### Performance overview
- Score: 10/10 (100%)
- Proficiency level: Expert / Master

### Summary
- Flawless understanding of retry logic, idempotency constraints, and architectural context.

### Key strengths
- Correct exponential backoff logic and the role of jitter to prevent thundering herd.
- Clear understanding that retries must be idempotent, especially under timeouts.
- Recognizes circuit breakers should override retries during outages.
- Correct transient error classification (HTTP 503).

### Areas for improvement
- None identified.

### Next level challenge
- Study retry budgets and adaptive concurrency control.
