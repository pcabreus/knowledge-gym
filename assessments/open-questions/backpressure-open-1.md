---
id: backpressure-open-1
topic: ../topics/system-design/backpressure.md
kind: open
difficulty: medium
timebox_minutes: 10
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Backpressure for API + Kafka + SQS

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
Design a backpressure strategy for a service that:
- Handles a synchronous HTTP API (user-facing),
- Processes Kafka events with a slow downstream dependency, and
- Runs an SQS worker for third‑party enrichment.
Explain the decision rules, concrete mechanisms in Go, and the observable signals that prove it works.

## Answer format (what “good” looks like)
- BLUF definition
- Decision rules (when to use / when not)
- API path: bounded concurrency + reject/degrade behavior
- Kafka consumer: pause/resume, `max.poll.records`, and lag control
- SQS worker: inflight limits + visibility timeout strategy
- Metrics + 1 failure mode and detection

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
- Unbounded queues instead of bounded concurrency.
- Retrying without a budget.
- Ignoring consumer lag or in-flight limits.
