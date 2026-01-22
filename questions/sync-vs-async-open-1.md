---
id: sync-vs-async-open-1
topic: ../topics/system-design/sync-vs-async-communication.md
kind: open
difficulty: medium
timebox_minutes: 8
created_at: 2026-01-22
updated_at: 2026-01-22
---

# Sync vs async decision

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
For a "thousands of communications per minute" contact platform, decide which flows should be synchronous vs asynchronous and why.

## Answer format (what “good” looks like)
- BLUF
- Decision rules and latency/consistency trade-offs
- 1 observability metric per style

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
- Making every flow async without consistency reasoning.
- No DLQ/idempotency mention for async flows.
