---
id: storage-compaction
title: "Storage Compaction"
type: concept
status: learning
importance: 45
difficulty: medium
tags: [database, storage]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Storage Compaction

## TL;DR (BLUF)
- Compaction reclaims space and reorganizes storage.
- Use it to control bloat and improve read performance.
- Trade-off: compaction is resource-intensive.

## Definition
**What it is:** A process that rewrites data to reduce fragmentation and reclaim space.
**Key terms:** compaction, fragmentation, rewrite.

## Why it matters
- It stabilizes storage usage and read performance.
- It can consume I/O and affect latency.

## Scope & Non-goals
**In scope:** compaction concepts and trade-offs.
**Out of scope / NOT solved by this:** database-specific compaction internals.

## Mental model / Intuition
- Think of compacting a disk to remove gaps.

## Decision rules (When to use / When not to use)
### Use it when
- Storage grows despite stable data size.
### Avoid it when
- You can tolerate bloat and want minimal maintenance.

## How I would use it (practical)
- **Context:** Large table with heavy updates.
- **Steps:** schedule compaction → monitor I/O → verify size reduction.
- **What success looks like:** reduced storage size and stable latency.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** reclaimed space and improved locality.
- **Cons / Risks:** I/O spikes and write slowdown.
### Alternatives
- **Vacuum/autovacuum:** lighter-weight cleanup in Postgres.
- **How to choose:** use compaction when lighter cleanup isn’t enough.

## Failure modes & Pitfalls
- Compaction during peak traffic causing latency spikes.

## Observability (How to detect issues)
- **Metrics:** disk I/O, latency, table size.
- **Logs:** compaction activity logs.
- **Alerts:** sustained I/O spikes during compaction.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Schedule during low-traffic windows
  - [ ] Monitor I/O impact

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Compaction during peak traffic.
  - **Why it’s bad:** causes latency spikes.
  - **Better approach:** use maintenance windows.

## Interview readiness
### “Explain it like I’m teaching”
- Compaction rewrites data to reclaim space and improve read locality. It’s useful but expensive, so schedule it carefully.

### Trap questions (with answers)
1) **Q:** Does compaction always improve performance?
   - **A:** not always; it can reduce performance during the process.
2) **Q:** Is compaction the same as vacuum?
   - **A:** no; vacuum is lighter and often doesn’t rewrite all data.
3) **Q:** Can compaction be continuous?
   - **A:** some systems do, but it still consumes resources.

### Quick self-check (5 items)
- [ ] I can define compaction.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain when to use it.
- [ ] I can connect it to maintenance windows.

## Links (NO duplication)
### Prerequisites
- [Storage fundamentals](storage-fundamentals.md)

### Related topics
- [PostgreSQL bloat](postgresql-bloat.md)

### Compare with
- [PostgreSQL vacuum & autovacuum](postgresql-vacuum-autovacuum.md) — lighter cleanup vs compaction.
