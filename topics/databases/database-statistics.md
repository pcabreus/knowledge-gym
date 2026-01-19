---
id: database-statistics
title: "Database Statistics"
type: concept
status: learning
importance: 45
difficulty: medium
tags: [database, performance]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Database Statistics

## TL;DR (BLUF)
- Statistics describe data distribution and guide query planners.
- Use them to improve plan quality and performance.
- Trade-off: stats collection adds overhead.

## Definition
**What it is:** Metadata about data distribution used for cost-based planning.
**Key terms:** histograms, cardinality, selectivity.

## Why it matters
- Bad stats lead to bad plans and slow queries.
- Stats help pick indexes and join order.

## Scope & Non-goals
**In scope:** general statistics concepts across databases.
**Out of scope / NOT solved by this:** engine-specific tuning.

## Mental model / Intuition
- Statistics are the planner’s “map” of the data.

## Decision rules (When to use / When not to use)
### Use it when
- You see plan regressions or slow queries after big data changes.
### Avoid it when
- Stats are already fresh and the issue is elsewhere.

## How I would use it (practical)
- **Context:** Query slowed after large load.
- **Steps:** update stats → re-check plan → monitor latency.
- **What success looks like:** improved plans and lower latency.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** better planning and performance.
- **Cons / Risks:** overhead and imperfect estimates.
### Alternatives
- **Plan hints:** when available, but risky.
- **How to choose:** rely on stats first, hints last.

## Failure modes & Pitfalls
- Assuming stats are exact.

## Observability (How to detect issues)
- **Metrics:** plan changes, query latency.
- **Logs:** stats update logs.
- **Alerts:** latency spikes after data shifts.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Update stats after large changes
  - [ ] Validate plan improvements

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Ignoring stats after bulk deletes.
  - **Why it’s bad:** planner uses stale info.
  - **Better approach:** refresh stats.

## Interview readiness
### “Explain it like I’m teaching”
- Database statistics are summaries of data distribution used by query planners. Keeping them fresh helps the planner choose efficient plans.

### Trap questions (with answers)
1) **Q:** Are statistics always accurate?
   - **A:** no; they’re estimates.
2) **Q:** Can stats hurt performance?
   - **A:** collecting them has overhead.
3) **Q:** Do stats remove the need for indexes?
   - **A:** no; they just help choose plans.

### Quick self-check (5 items)
- [ ] I can define database statistics.
- [ ] I can explain why they matter.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can relate stats to planning.

## Links (NO duplication)
### Prerequisites
- [Selectivity](selectivity.md)

### Related topics
- [Query planning](query-planning.md)

### Compare with
- [PostgreSQL statistics & ANALYZE](postgresql-stats-and-analyze.md) — Postgres specifics.
