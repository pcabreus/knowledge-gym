---
id: performance-basics
title: "Performance Basics"
type: concept
status: learning
importance: 40
difficulty: easy
tags: [system-design, performance]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Performance Basics

## TL;DR (BLUF)
- Performance is about latency, throughput, and resource usage.
- Use metrics and profiling to find bottlenecks.
- Trade-off: optimizations often increase complexity.

## Definition
**What it is:** Measuring and optimizing system speed and efficiency.
**Key terms:** latency, throughput, p95, saturation.

## Why it matters
- Performance impacts user experience and cost.
- Premature optimization can waste effort.

## Scope & Non-goals
**In scope:** core performance metrics and trade-offs.
**Out of scope / NOT solved by this:** low-level hardware tuning.

## Mental model / Intuition
- Optimize the bottleneck, not everything.

## Decision rules (When to use / When not to use)
### Use it when
- You have measurable latency or throughput issues.
### Avoid it when
- You lack metrics or clear bottlenecks.

## How I would use it (practical)
- **Context:** API latency spike.
- **Steps:** measure p95 → isolate bottleneck → optimize → validate.
- **What success looks like:** lower p95 and stable throughput.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** faster systems.
- **Cons / Risks:** added complexity.
### Alternatives
- **Scaling up:** sometimes simpler.
- **How to choose:** measure before optimizing.

## Failure modes & Pitfalls
- Optimizing the wrong component.

## Observability (How to detect issues)
- **Metrics:** latency p95, error rate, saturation.
- **Logs:** slow operation logs.
- **Alerts:** sustained p95 spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define SLIs
  - [ ] Measure before/after

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Premature optimization.
  - **Why it’s bad:** wasted effort and complexity.
  - **Better approach:** optimize after measuring.

## Interview readiness
### “Explain it like I’m teaching”
- Performance is about how fast and efficiently a system responds. You measure latency and throughput, find the bottleneck, then optimize.

### Trap questions (with answers)
1) **Q:** Is average latency enough?
   - **A:** no; p95/p99 matter.
2) **Q:** Should you optimize without metrics?
   - **A:** no; you risk optimizing the wrong thing.
3) **Q:** Is scaling always better than optimization?
   - **A:** not always; sometimes optimization is cheaper.

### Quick self-check (5 items)
- [ ] I can define latency and throughput.
- [ ] I can explain p95.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain bottleneck thinking.

## Links (NO duplication)
### Prerequisites
- [Observability basics](../operations/observability-basics.md)

### Related topics
- [Slow query diagnosis](../databases/slow-query-diagnosis.md)

### Compare with
- [Capacity planning](../operations/capacity-planning.md)
