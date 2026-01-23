---
id: service-level-indicator
title: "Service Level Indicator (SLI)"
type: concept
status: learning
importance: 70
difficulty: easy
tags: [reliability, observability]
canonical: true
aliases: []
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Service Level Indicator (SLI)

## TL;DR (BLUF)
- An SLI is a measurable signal of service health (e.g., availability, latency, error rate).
- Use SLIs to quantify user experience and drive SLOs.
- Trade-off: picking the wrong SLI gives false confidence.

## Definition
**What it is:** A quantitative metric that reflects how users experience a service.  
**Key terms:** availability, latency, error rate, good events/total events.

## Why it matters
- SLIs turn reliability into measurable signals.
- Bad SLIs lead to misaligned priorities and noisy alerts.

## Scope & Non-goals
**In scope:** selecting and measuring user-focused reliability signals.  
**Out of scope / NOT solved by this:** setting targets (see [Service Level Objective (SLO)](service-level-objective.md)) or contractual promises (see [Service Level Agreement (SLA)](service-level-agreement.md)).

## Mental model / Intuition
- “What number tells me whether users are happy?”

## Decision rules (When to use / When not to use)
### Use it when
- You need a single, user-centric signal for reliability decisions.
- You are defining or monitoring [Service Level Objectives (SLOs)](service-level-objective.md).

### Avoid it when
- You only track internal metrics that don’t map to user experience.

## How I would use it (practical)
- **Context:** A public API with latency complaints.
- **Steps:**
  1) Define a latency SLI: % of requests under 300ms.
  2) Define availability SLI: % of successful responses.
  3) Track SLIs by endpoint and tier.
- **What success looks like:** SLIs align with user-perceived performance and guide SLOs.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** clear, user-centric measurement.
- **Cons / Risks:** oversimplification if SLIs ignore critical segments.
### Alternatives
- **Composite SLIs:** combine multiple signals (latency + error rate).
- **How to choose:** start with a small set, then refine by customer segments.

## Failure modes & Pitfalls
- Using system metrics (CPU) as SLIs instead of user signals.
- Aggregating across endpoints and hiding poor performance in critical paths.

## Observability (How to detect issues)
- **Metrics:** good events/total events, p95 latency, error rate.
- **Logs:** failed requests with endpoint and status.
- **Traces:** slow spans in critical paths.
- **Alerts:** SLI drop below threshold (burn alert).

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Choose user-centric signals (latency, errors, availability)
  - [ ] Define “good” events precisely
  - [ ] Segment by tier/endpoint
  - [ ] Validate SLI aligns with complaints
- **Security / Compliance notes**
  - Ensure SLIs exclude sensitive payloads
- **Performance notes**
  - Use low-cardinality labels
- **Operational notes**
  - Revisit SLIs after product changes

## Mini example (if applicable)
```text
SLI_latency = % of requests under 300ms over 28 days
SLI_availability = % of 2xx/3xx responses over total
```

## Common anti-patterns
- **Anti-pattern:** “CPU < 70% is our SLI.”
  - **Why it’s bad:** CPU is not user experience.
  - **Better approach:** use latency/error rate as SLIs.

## Interview readiness
### “Explain it like I’m teaching”
- An SLI is a number that reflects user experience, like the percent of requests under 300ms or the percent of successful responses.

### Trap questions (with answers)
1) **Q:** Is CPU usage an SLI?
   - **A:** No. It’s an internal signal, not a user-facing outcome.
2) **Q:** Can you have multiple SLIs?
   - **A:** Yes. Many services track latency and availability SLIs.
3) **Q:** Are SLIs targets?
   - **A:** No. Targets are SLOs built on top of SLIs.

### Quick self-check (5 items)
- [ ] I can define an SLI precisely.
- [ ] I can name at least 2 common SLIs.
- [ ] I can explain “good events/total events.”
- [ ] I can link SLIs to SLOs.
- [ ] I can name a failure mode in SLI selection.

## Links (NO duplication)
### Prerequisites
- [Observability basics](observability-basics.md)

### Related topics
- [Service Level Objective (SLO)](service-level-objective.md)
- [Service Level Agreement (SLA)](service-level-agreement.md)
- [Error budgets](error-budgets.md)

### Compare with
- [Service Level Objective (SLO)](service-level-objective.md) — SLI is the metric; SLO is the target.

## Notes / Inbox (optional)
- N/A
