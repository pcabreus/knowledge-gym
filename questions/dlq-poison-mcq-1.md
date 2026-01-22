---
id: dlq-poison-mcq-1
topic: ../topics/operations/dlq-and-poison-messages.md
kind: mcq
difficulty: medium
timebox_minutes: 4
created_at: 2026-01-22
updated_at: 2026-01-22
studied: true
studied_at: 2026-01-22
score: 9/10
---

# DLQ and poison messages

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
Which scenario best justifies routing a message to a DLQ (dead-letter queue) immediately?

## Answer format (what “good” looks like)
- Identify permanent failure
- Avoid retry storms

## Answer key / Rubric
### Correct answer (for MCQ)
- Correct: C
- Why correct: A schema validation error that can’t be parsed is permanent for that consumer; retries won’t help, so DLQ is appropriate.
- Why A is wrong: A 503 is transient and should be retried with backoff.
- Why B is wrong: A temporary network timeout is transient and should be retried.
- Why C is wrong: N/A
- Why D is wrong: “Send all messages to DLQ first” is absurd and defeats the pipeline.

A) Provider returns HTTP 503 for 2 minutes.
B) A network timeout occurs once.
C) The payload fails schema validation due to missing required fields.
D) Send all messages to the DLQ to keep the main queue empty.

## Common mistakes
- Treating transient failures as poison.
- Replaying DLQ without idempotency.

## Evaluation (latest)
### Performance overview
- Score: 9/10 (90%)
- Proficiency level: Advanced / Expert

### Summary
- Strong operational knowledge of DLQ placement within retry and reliability architecture.

### Key strengths
- Idempotency as the safeguard when replaying messages to avoid side effects.
- Interpreting DLQ spikes after deployments as likely consumer bugs.
- Using retries for transient failures like OOM instead of immediate DLQ.
- Awareness of FIFO head-of-line blocking risks and circuit breaker protection.
- Correct use of redrive policies and maxReceiveCount.

### Area for improvement
- Distinguish transient vs. permanent failures: network timeouts are transient and should be retried with backoff. Immediate DLQ is for permanent failures (e.g., schema validation errors).
