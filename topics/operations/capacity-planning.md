---
id: capacity-planning
title: "Capacity Planning"
type: skill
status: learning
importance: 40
difficulty: medium
tags: [operations, scalability]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Capacity Planning

## TL;DR (BLUF)
- Capacity planning ensures systems can handle future load.
- Use it to prevent outages and overprovisioning.
- Trade-off: inaccurate forecasts cause waste or risk.

## Definition
**What it is:** Estimating resource needs based on load projections.
**Key terms:** headroom, forecast, utilization.

## Why it matters
- It avoids sudden performance degradation.
- It balances cost and reliability.

## Scope & Non-goals
**In scope:** planning resource capacity.
**Out of scope / NOT solved by this:** real-time autoscaling logic.

## Mental model / Intuition
- Plan for growth before you hit limits.

## Decision rules (When to use / When not to use)
### Use it when
- You have predictable growth or seasonal spikes.
### Avoid it when
- Your system is fully autoscaled and stable (rare).

## How I would use it (practical)
- **Context:** Increasing traffic forecasts.
- **Steps:** estimate demand → check utilization → add headroom.
- **What success looks like:** no capacity-related incidents.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** stability and preparedness.
- **Cons / Risks:** overprovisioning.
### Alternatives
- **Autoscaling:** reduces manual planning.
- **How to choose:** combine autoscaling with planning for limits.

## Failure modes & Pitfalls
- Relying on outdated usage trends.

## Observability (How to detect issues)
- **Metrics:** CPU, memory, I/O utilization.
- **Logs:** scaling events.
- **Alerts:** sustained high utilization.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Monitor utilization trends
  - [ ] Set headroom targets

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Planning for average load only.
  - **Why it’s bad:** peaks cause outages.
  - **Better approach:** plan for p95/p99.

## Interview readiness
### “Explain it like I’m teaching”
- Capacity planning predicts resource needs so you don’t hit limits. It balances cost with reliability by adding headroom for growth.

### Trap questions (with answers)
1) **Q:** Is autoscaling enough?
   - **A:** not always; hard limits still exist.
2) **Q:** Should you plan on average load?
   - **A:** no; plan for peaks.
3) **Q:** Can you skip planning if traffic is low?
   - **A:** not if you expect growth.

### Quick self-check (5 items)
- [ ] I can define capacity planning.
- [ ] I can explain headroom.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can identify a monitoring signal.

## Links (NO duplication)
### Prerequisites
- N/A

### Related topics
- [Performance basics](../system-design/performance-basics.md)

### Compare with
- [Reliability basics](reliability-basics.md) — capacity vs stability.
