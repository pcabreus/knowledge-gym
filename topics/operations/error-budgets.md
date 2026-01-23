---
id: error-budgets
title: "Error Budgets"
type: concept
status: learning
importance: 70
difficulty: medium
tags: [reliability, operations]
canonical: true
aliases: []
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Error Budgets

## TL;DR (BLUF)
- Error budgets quantify how much unreliability you can “spend” while meeting SLOs.
- Use them to balance reliability with release velocity.
- Trade-off: strict budgets slow change; loose budgets risk user pain.

## Definition
**What it is:** The allowed amount of SLO violations within a time window, derived from the SLO target.  
**Key terms:** burn rate, budget window, SLO compliance.

## Why it matters
- Turns reliability into a decision tool for shipping vs pausing changes.
- Prevents over-reacting to single incidents or under-reacting to chronic issues.

## Scope & Non-goals
**In scope:** policy decisions tied to SLO burn.  
**Out of scope / NOT solved by this:** setting the SLO itself (see [Service Level Objective (SLO)](service-level-objective.md)).

## Mental model / Intuition
- “You have a budget of failure. Spend it wisely.”

## Decision rules (When to use / When not to use)
### Use it when
- You want a clear go/no-go signal for releases.
- You need to trade off reliability vs feature velocity.

### Avoid it when
- You cannot measure the SLI accurately.

## How I would use it (practical)
- **Context:** An API with a 99.9% availability SLO over 30 days.
- **Steps:**
  1) Calculate error budget: 0.1% of requests can fail.
  2) Monitor burn rate (fast vs slow).
  3) Freeze releases if the budget is exhausted.
- **What success looks like:** stable reliability and predictable release policy.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** objective release gating.
- **Cons / Risks:** can over-constrain delivery if SLOs are too tight.
### Alternatives
- **Qualitative gating:** subjective go/no-go decisions.
- **How to choose:** use budgets when you can measure SLIs reliably.

## Failure modes & Pitfalls
- SLOs set unrealistically high → budget exhausted immediately.
- Ignoring fast-burn alerts → late reaction to incidents.

## Observability (How to detect issues)
- **Metrics:** burn rate, remaining budget, SLI compliance.
- **Logs:** SLO breach events.
- **Traces:** root causes of budget burn.
- **Alerts:** fast burn (e.g., 2x) and slow burn alerts.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define SLO and time window
  - [ ] Calculate budget and burn alerts
  - [ ] Define release policy tied to budget
- **Security / Compliance notes**
  - N/A
- **Performance notes**
  - N/A
- **Operational notes**
  - Revisit budgets after major incidents

## Mini example (if applicable)
```text
SLO = 99.9% over 30 days
Error budget = 0.1% of requests = 43.2 minutes of downtime/month
```

## Common anti-patterns
- **Anti-pattern:** “Ignore budgets during launches.”
  - **Why it’s bad:** reliability debt accumulates quickly.
  - **Better approach:** plan reliability work into the roadmap.

## Interview readiness
### “Explain it like I’m teaching”
- Error budgets are the allowed amount of failure within an SLO window, used to decide when to ship or slow down.

### Trap questions (with answers)
1) **Q:** Are error budgets the same as SLOs?
   - **A:** No. The budget is derived from the SLO target.
2) **Q:** Do error budgets replace incident response?
   - **A:** No. They guide release decisions, not recovery.
3) **Q:** If the budget is exhausted, do you stop all work?
   - **A:** Usually you pause risky changes and focus on reliability work.

### Quick self-check (5 items)
- [ ] I can define error budgets precisely.
- [ ] I can calculate a simple budget.
- [ ] I can describe burn rate alerts.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.

## Links (NO duplication)
### Prerequisites
- [Service Level Objective (SLO)](service-level-objective.md)

### Related topics
- [Service Level Indicator (SLI)](service-level-indicator.md)
- [Service Level Agreement (SLA)](service-level-agreement.md)
- [Reliability basics](reliability-basics.md)

### Compare with
- [Service Level Objective (SLO)](service-level-objective.md) — SLO is the target; budget is the allowed error.

## Notes / Inbox (optional)
- N/A
