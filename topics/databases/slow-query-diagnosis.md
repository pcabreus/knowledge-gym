---
id: slow-query-diagnosis
title: "Slow Query Diagnosis"
type: skill
status: learning
importance: 55
difficulty: medium
tags: [database, performance]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Slow Query Diagnosis

## TL;DR (BLUF)
- Diagnose slow queries by combining EXPLAIN, statistics, and workload context.
- Use a repeatable checklist to avoid guesswork.
- Trade-off: deep analysis takes time; quick fixes may be wrong.

## Definition
**What it is:** A systematic process for identifying and fixing slow database queries.
**Key terms:** EXPLAIN, selectivity, indexes, statistics.

## Why it matters
- Slow queries dominate latency and cost.
- Incorrect fixes can worsen writes or other queries.

## Scope & Non-goals
**In scope:** diagnosis workflow for common slow-query causes.
**Out of scope / NOT solved by this:** database-wide capacity planning.

## Mental model / Intuition
- Treat slow queries as a symptom; find the root cause before changing indexes.

## Decision rules (When to use / When not to use)
### Use it when
- p95 latency spikes or SLAs are missed.
### Avoid it when
- You haven’t verified the query is the bottleneck.

## How I would use it (practical)
- **Context:** A endpoint with high latency.
- **Steps:** capture query → run EXPLAIN → check scans/selectivity → validate indexes → measure impact.
- **What success looks like:** latency reduced without regressions.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** targeted fixes with measurable impact.
- **Cons / Risks:** time-consuming and requires expertise.
### Alternatives
- **Caching:** sometimes faster than deep query changes.
- **How to choose:** diagnose first; only cache if query is stable and safe.

## Failure modes & Pitfalls
- Adding indexes without validating plans.
- Ignoring data distribution changes.

## Observability (How to detect issues)
- **Metrics:** slow query count, p95 latency, CPU and I/O.
- **Logs:** slow query logs.
- **Alerts:** sustained slow query spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Identify slow queries
  - [ ] Run EXPLAIN
  - [ ] Validate selectivity
  - [ ] Measure before/after

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Adding indexes without understanding the query.
  - **Why it’s bad:** can worsen writes and not help reads.
  - **Better approach:** analyze plan and selectivity first.

## Interview readiness
### “Explain it like I’m teaching”
- Slow query diagnosis is a disciplined process: capture the query, inspect the plan, check selectivity, adjust indexes, and measure results. It avoids guesswork.

### Trap questions (with answers)
1) **Q:** Does an index always fix a slow query?
   - **A:** no; it depends on selectivity and query shape.
2) **Q:** Is EXPLAIN enough by itself?
   - **A:** no; you also need data distribution and workload context.
3) **Q:** Should you always cache instead?
   - **A:** no; caching can hide underlying issues.

### Quick self-check (5 items)
- [ ] I can describe a diagnosis workflow.
- [ ] I can explain why selectivity matters.
- [ ] I can name a pitfall.
- [ ] I can describe a success metric.
- [ ] I can list tools used.

## Links (NO duplication)
### Prerequisites
- [EXPLAIN](explain.md)
- [Selectivity](selectivity.md)

### Related topics
- [Index](index.md)

### Compare with
- [Caching strategy](../system-design/caching-strategy.md)
