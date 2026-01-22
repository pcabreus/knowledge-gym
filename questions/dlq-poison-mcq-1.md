---
id: dlq-poison-mcq-1
topic: ../topics/operations/dlq-and-poison-messages.md
kind: mcq
difficulty: medium
timebox_minutes: 4
created_at: 2026-01-22
updated_at: 2026-01-22
---

# DLQ and poison messages

## Prompt
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
