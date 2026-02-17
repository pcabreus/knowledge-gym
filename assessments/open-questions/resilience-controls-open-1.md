---
id: resilience-controls-open-1
topic: ../topics/operations/load-shedding.md
kind: open
difficulty: medium
timebox_minutes: 12
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Choosing between backpressure, load shedding, circuit breaker, and bulkheads

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
You run an API with a mix of critical and non-critical endpoints. A flaky downstream dependency causes spikes in latency and timeouts. Traffic surges happen during peak hours. Explain how you would use **backpressure**, **load shedding**, **circuit breakers**, and **bulkheads** together. Clarify what each one protects, where it applies (edge vs internal), and what signals would trigger each.

## Answer format (what “good” looks like)
- BLUF definition of each pattern
- Decision rules (when to use / when not)
- Concrete placement (edge vs internal) and signals
- Trade-offs and failure modes
- One example of how they interact (defense in depth)

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
- Treating backpressure and load shedding as the same.
- Using breakers without timeouts.
- Ignoring isolation for noisy neighbors.
