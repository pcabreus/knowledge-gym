---
id: service-level-agreement
title: "Service Level Agreement (SLA)"
type: concept
status: learning
importance: 70
difficulty: easy
tags: [reliability, contracts]
canonical: true
aliases: []
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Service Level Agreement (SLA)

## TL;DR (BLUF)
- An SLA is a contractual promise to customers about service levels.
- Use SLAs to set external expectations and penalties.
- Trade-off: stricter SLAs increase operational cost and risk.

## Definition
**What it is:** A contract between provider and customer defining minimum service levels, reporting, and remedies/penalties if missed.  
**Key terms:** contract, penalty, service credits, reporting window.

## Why it matters
- SLAs drive legal and financial obligations.
- Misaligned SLAs can create business risk and customer churn.

## Scope & Non-goals
**In scope:** external commitments and remedies.  
**Out of scope / NOT solved by this:** internal targets (see [Service Level Objective (SLO)](service-level-objective.md)).

## Mental model / Intuition
- “What we guarantee to customers, with consequences.”

## Decision rules (When to use / When not to use)
### Use it when
- You sell a service with explicit reliability commitments.
- You need clear escalation and compensation rules.

### Avoid it when
- You cannot measure or report reliably.

## How I would use it (practical)
- **Context:** A paid API product with enterprise customers.
- **Steps:**
  1) Set internal SLOs higher than the SLA.
  2) Define reporting windows and exclusions.
  3) Define service credits for breaches.
- **What success looks like:** customers understand expectations; internal teams keep enough buffer.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** clear customer expectations; accountability.
- **Cons / Risks:** legal exposure, higher reliability cost.
### Alternatives
- **Best-effort service:** no contractual guarantees.
- **How to choose:** if customers need guarantees, formalize an SLA.

## Failure modes & Pitfalls
- SLA tighter than internal SLO → constant breaches.
- Vague exclusions or unclear reporting windows.

## Observability (How to detect issues)
- **Metrics:** SLA compliance %, uptime in contract window.
- **Logs:** audit trail for outages and exclusions.
- **Traces:** N/A
- **Alerts:** risk of SLA breach (burn alerts).

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define SLA scope and exclusions
  - [ ] Align internal SLOs above SLA
  - [ ] Define reporting windows
  - [ ] Specify penalties/remedies
- **Security / Compliance notes**
  - Ensure contractual metrics are auditable
- **Performance notes**
  - N/A
- **Operational notes**
  - Review SLA terms annually

## Mini example (if applicable)
```text
SLA: 99.5% monthly availability, service credits if below
```

## Common anti-patterns
- **Anti-pattern:** “Promise 99.99% without capacity.”
  - **Why it’s bad:** guarantees breaches and financial risk.
  - **Better approach:** set internal SLOs first, then set SLA.

## Interview readiness
### “Explain it like I’m teaching”
- An SLA is an external contractual promise with remedies if missed, and it should be looser than internal SLOs.

### Trap questions (with answers)
1) **Q:** Are SLAs the same as SLOs?
   - **A:** No. SLA is contractual; SLO is internal.
2) **Q:** Should SLA and SLO be identical?
   - **A:** No. SLOs should be tighter to provide buffer.
3) **Q:** Do SLAs include all incidents?
   - **A:** Not necessarily; exclusions must be explicit.

### Quick self-check (5 items)
- [ ] I can define an SLA precisely.
- [ ] I can explain how it differs from SLO.
- [ ] I can name a trade-off.
- [ ] I can give a concrete example.
- [ ] I can name a pitfall.

## Links (NO duplication)
### Prerequisites
- [Service Level Objective (SLO)](service-level-objective.md)

### Related topics
- [Service Level Indicator (SLI)](service-level-indicator.md)
- [Error budgets](error-budgets.md)

### Compare with
- [Service Level Objective (SLO)](service-level-objective.md) — internal target vs contractual promise.

## Notes / Inbox (optional)
- N/A
