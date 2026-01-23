---
id: service-level-objective
title: "Service Level Objective (SLO)"
type: concept
status: learning
importance: 75
difficulty: easy
tags: [reliability, observability]
canonical: true
aliases: []
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Service Level Objective (SLO)

## TL;DR (BLUF)
- An SLO is the target level of service you aim to deliver, measured via SLIs.
- Use SLOs to balance reliability and velocity with an error budget.
- Trade-off: tighter SLOs increase cost and slow change cadence.

## Definition
**What it is:** A reliability target (e.g., 99.9% availability, 95% of requests under 300ms) defined over a time window.  
**Key terms:** target, time window, error budget, burn rate.

## Why it matters
- SLOs create shared expectations between product and engineering.
- They guide prioritization when reliability slips.

## Scope & Non-goals
**In scope:** target setting for user-facing reliability.  
**Out of scope / NOT solved by this:** contractual promises (see [Service Level Agreement (SLA)](service-level-agreement.md)).

## Mental model / Intuition
- “How good do we need to be for users to be satisfied?”

## Decision rules (When to use / When not to use)
### Use it when
- You need clear reliability targets tied to user experience.
- You want to manage trade-offs with [Error budgets](error-budgets.md).

### Avoid it when
- You cannot measure the SLI reliably.

## How I would use it (practical)
- **Context:** A B2B API with enterprise customers.
- **Steps:**
  1) Define SLIs for latency and availability.
  2) Set SLOs (e.g., 99.9% availability, 95% < 300ms).
  3) Use error budget burn to manage release risk.
- **What success looks like:** decisions align with SLO compliance and customer expectations.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** clear targets and decision rules.
- **Cons / Risks:** over-tight SLOs increase cost and reduce velocity.
### Alternatives
- **Best-effort reliability:** for low-stakes internal tools.
- **How to choose:** base SLOs on user tolerance and business impact.

## Failure modes & Pitfalls
- Setting “aspirational” SLOs without capacity to meet them.
- Using averages instead of percentiles for latency SLOs.

## Observability (How to detect issues)
- **Metrics:** SLI compliance %, error budget burn rate.
- **Logs:** SLO breaches with context.
- **Traces:** slowest paths causing violations.
- **Alerts:** burn rate alerts (fast/slow).

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define SLIs and time windows
  - [ ] Set realistic targets with stakeholders
  - [ ] Monitor error budget burn
  - [ ] Tie release policy to burn rate
- **Security / Compliance notes**
  - N/A
- **Performance notes**
  - Use percentiles for latency targets
- **Operational notes**
  - Review SLOs quarterly

## Mini example (if applicable)
```text
SLO: 99.9% availability over 30 days
SLO: 95% of requests under 300ms over 7 days
```

## Common anti-patterns
- **Anti-pattern:** “Set 100% uptime SLO.”
  - **Why it’s bad:** unrealistic; it removes the error budget needed for change.
  - **Better approach:** set a realistic target with a clear budget.

## Interview readiness
### “Explain it like I’m teaching”
- An SLO is a target for reliability measured via SLIs, and it drives error budgets and release decisions.

### Trap questions (with answers)
1) **Q:** Are SLOs the same as SLAs?
   - **A:** No. SLOs are internal targets; SLAs are external contracts.
2) **Q:** Should SLOs be tighter than SLAs?
   - **A:** Usually yes, to provide buffer before contractual penalties.
3) **Q:** Can you have multiple SLOs?
   - **A:** Yes. Many services track both availability and latency SLOs.

### Quick self-check (5 items)
- [ ] I can define an SLO precisely.
- [ ] I can state how SLOs use SLIs.
- [ ] I can explain error budgets.
- [ ] I can describe a trade-off.
- [ ] I can name a failure mode.

## Links (NO duplication)
### Prerequisites
- [Service Level Indicator (SLI)](service-level-indicator.md)

### Related topics
- [Service Level Agreement (SLA)](service-level-agreement.md)
- [Error budgets](error-budgets.md)
- [Reliability basics](reliability-basics.md)

### Compare with
- [Service Level Agreement (SLA)](service-level-agreement.md) — SLO is internal; SLA is contractual.

## Notes / Inbox (optional)
- N/A
