---
id: reliability-basics
title: "Reliability Basics"
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

# Reliability Basics

## TL;DR (BLUF)
- Reliability is about consistent, predictable system behavior.
- Use [Service Level Indicators (SLIs)](service-level-indicator.md) and [Service Level Objectives (SLOs)](service-level-objective.md) to measure and manage reliability.
- Trade-off: higher reliability often costs more.

## Definition
**What it is:** The likelihood a system performs correctly over time.
**Key terms:** [Service Level Indicator (SLI)](service-level-indicator.md), [Service Level Objective (SLO)](service-level-objective.md), [Error budgets](error-budgets.md).

## Why it matters
- Reliability affects user trust and business outcomes.

## Scope & Non-goals
**In scope:** correctness-over-time concepts and SLO framing.
**Out of scope / NOT solved by this:** availability-only targets.

## Mental model / Intuition
- Reliability is consistency over time, not perfection.

## Decision rules (When to use / When not to use)
### Use it when
- You set uptime or latency targets.
### Avoid it when
- The system is non-critical and experimental.

## How I would use it (practical)
- **Context:** API uptime goals.
- **Steps:** define SLO → monitor → act on [Error budgets](error-budgets.md).
- **What success looks like:** stable uptime and clear trade-offs.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** predictable behavior.
- **Cons / Risks:** higher cost and slower change cadence.
### Alternatives
- **Best-effort reliability:** for non-critical services.
- **How to choose:** match reliability targets to business impact.

## Failure modes & Pitfalls
- Setting unrealistic SLOs.

## Observability (How to detect issues)
- **Metrics:** error rate, latency p95.
- **Logs:** error logs.
- **Alerts:** SLO burn.

## Implementation notes (if applicable)
- **Checklist**
   - [ ] Define [Service Level Indicators (SLIs)](service-level-indicator.md)
   - [ ] Set [Service Level Objectives (SLOs)](service-level-objective.md)

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Chasing 100% uptime.
  - **Why it’s bad:** impractical and expensive.
  - **Better approach:** set realistic SLOs.

## Interview readiness
### “Explain it like I’m teaching”
- Reliability is the ability of a system to behave correctly over time. Use SLIs/SLOs and [Error budgets](error-budgets.md) to manage it.

### Trap questions (with answers)
1) **Q:** Is 100% uptime achievable?
   - **A:** no; it’s impractical.
2) **Q:** Are SLOs only for ops teams?
   - **A:** no; they guide engineering trade-offs.
3) **Q:** Does reliability mean zero errors?
   - **A:** no; it’s about acceptable error rates.

### Quick self-check (5 items)
- [ ] I can define reliability.
- [ ] I can explain SLIs/SLOs.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain [Error budgets](error-budgets.md).

## Links (NO duplication)
### Prerequisites
- N/A

### Related topics
- [Networking basics](networking-basics.md)
- [Availability basics](availability-basics.md)
- [Service Level Indicator (SLI)](service-level-indicator.md)
- [Service Level Objective (SLO)](service-level-objective.md)
- [Service Level Agreement (SLA)](service-level-agreement.md)
- [Error budgets](error-budgets.md)

### Compare with
- [Availability basics](availability-basics.md)
