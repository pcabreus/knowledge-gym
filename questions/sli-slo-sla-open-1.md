---
id: sli-slo-sla-open-1
topic: ../topics/operations/service-level-objective.md
kind: open
difficulty: medium
timebox_minutes: 10
created_at: 2026-01-23
updated_at: 2026-01-23
---

# SLI/SLO/SLA and error budgets in practice

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
You own a paid API used by enterprise customers. Define your **SLIs**, propose **SLOs**, and explain what goes into the **SLA**. Then describe how you would calculate and use the **error budget** for release decisions.

## Answer format (what “good” looks like)
- BLUF definitions of SLI/SLO/SLA
- 2–3 concrete SLIs (latency + availability)
- SLO targets with windows
- SLA terms (reporting window, exclusions, credits)
- Error budget calculation and release policy

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
- Confusing SLOs with SLAs.
- No error budget policy tied to releases.
- SLIs that don’t reflect user experience.
