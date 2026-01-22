---
id: microservices-design-open-1
topic: ../topics/architecture/microservices-design-basics.md
kind: design
difficulty: hard
timebox_minutes: 12
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Microservices boundaries and communication

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
Design microservice boundaries for a debt-collection platform and justify sync vs async communication between them. Include versioning and failure isolation.

## Answer format (what “good” looks like)
- BLUF boundaries + rationale
- Communication style per interaction
- Versioning policy + failure isolation

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
- Splitting by tech layers.
- No observability or error-budget thinking.
