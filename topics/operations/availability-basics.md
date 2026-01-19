---
id: availability-basics
title: "Availability Basics"
type: concept
status: learning
importance: 40
difficulty: easy
tags: [operations, reliability]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Availability Basics

## TL;DR (BLUF)
- Availability measures how often a system is operational.
- Use it to set uptime targets and SLOs.
- Trade-off: higher availability costs more.

## Definition
**What it is:** The proportion of time a system is functioning and accessible.
**Key terms:** uptime, downtime, SLO.

## Why it matters
- Availability affects user trust and revenue.

## Scope & Non-goals
**In scope:** uptime targets and availability trade-offs.
**Out of scope / NOT solved by this:** correctness/reliability semantics.

## Mental model / Intuition
- Availability is about time, not correctness.

## Decision rules (When to use / When not to use)
### Use it when
- You set uptime requirements.
### Avoid it when
- The service is non-critical and experimental.

## How I would use it (practical)
- **Context:** Defining uptime goals for an API.
- **Steps:** choose SLO → measure uptime → track error budgets.
- **What success looks like:** stable uptime within SLO.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** clear target.
- **Cons / Risks:** higher cost.
### Alternatives
- **Best-effort availability:** for non-critical systems.
- **How to choose:** align targets with business impact.

## Failure modes & Pitfalls
- Measuring availability without accounting for user impact.

## Observability (How to detect issues)
- **Metrics:** uptime percentage, error rate.
- **Logs:** outage reports.
- **Alerts:** SLO burn rate.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define SLOs
  - [ ] Measure uptime accurately

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Chasing five 9s without need.
  - **Why it’s bad:** huge cost.
  - **Better approach:** set realistic targets.

## Interview readiness
### “Explain it like I’m teaching”
- Availability is the percentage of time your system is up and serving requests. Higher availability costs more, so align goals with business needs.

### Trap questions (with answers)
1) **Q:** Is availability the same as reliability?
   - **A:** no; reliability includes correctness and consistency.
2) **Q:** Does more redundancy always improve availability?
   - **A:** not if it adds complexity and failure modes.
3) **Q:** Is 100% availability realistic?
   - **A:** no; aim for realistic SLOs.

### Quick self-check (5 items)
- [ ] I can define availability.
- [ ] I can explain trade-offs.
- [ ] I can name a pitfall.
- [ ] I can describe a monitoring signal.
- [ ] I can relate to reliability basics.

## Links (NO duplication)
### Prerequisites
- N/A

### Related topics
- [Reliability basics](reliability-basics.md)

### Compare with
- [Reliability basics](reliability-basics.md) — uptime vs correctness.
