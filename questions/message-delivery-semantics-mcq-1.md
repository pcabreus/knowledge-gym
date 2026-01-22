---
id: message-delivery-semantics-mcq-1
topic: ../topics/architecture/message-delivery-semantics.md
kind: mcq
difficulty: hard
timebox_minutes: 5
created_at: 2026-01-22
updated_at: 2026-01-22
studied: true
studied_at: 2026-01-22
---

# Message delivery semantics: choosing guarantees

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
In a pipeline where a consumer writes to Postgres and calls a third-party API, which delivery guarantee is most realistic end-to-end and what design choice makes it safe?

## Answer format (what “good” looks like)
- Pick a delivery semantic and justify it
- Mention idempotency and where it must be enforced

## Answer key / Rubric
### Correct answer (for MCQ)
- Correct: B
- Why correct: End-to-end exactly-once is unrealistic across broker + DB + third-party boundaries. The practical guarantee is at-least-once, made safe by idempotent consumers and idempotency keys for external calls.
- Why A is wrong: At-most-once risks silent loss and does not match the requirement for correctness with DB + API side effects.
- Why B is wrong: N/A
- Why C is wrong: Kafka EOS does not cover side effects outside Kafka; it is scoped and not end-to-end.
- Why D is wrong: “Exactly-once by turning off retries” is absurd; it increases loss and doesn’t guarantee single processing.

A) At-most-once; accept losses because retries add risk.
B) At-least-once; make the consumer idempotent and use idempotency keys.
C) Exactly-once; enable Kafka EOS and assume downstream side effects follow.
D) Exactly-once by disabling retries and timeouts.

## Common mistakes
- Confusing broker guarantees with end-to-end side effects.
- Ignoring idempotency for third-party APIs.
