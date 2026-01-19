---
id: postgresql-vacuum-autovacuum
title: "PostgreSQL Vacuum & Autovacuum"
type: concept
status: learning
importance: 60
difficulty: medium
tags: [database, postgresql, maintenance]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# PostgreSQL Vacuum & Autovacuum

## TL;DR (BLUF)
- Vacuum reclaims space from dead tuples; autovacuum does it automatically.
- Use it to control bloat and maintain performance.
- Trade-off: aggressive vacuum can add write overhead.

## Definition
**What it is:** A maintenance process that cleans dead tuples and updates visibility maps.
**Key terms:** dead tuples, autovacuum, freeze.

## Why it matters
- Without vacuum, tables and indexes bloat and queries slow down.
- Autovacuum settings directly impact performance stability.

## Scope & Non-goals
**In scope:** vacuum purpose, autovacuum tuning basics.
**Out of scope / NOT solved by this:** manual disk maintenance beyond vacuum.

## Mental model / Intuition
- Vacuum is the janitor that cleans old row versions.

## Decision rules (When to use / When not to use)
### Use it when
- You see growing dead tuples or table bloat.
### Avoid it when
- You need zero maintenance overhead (not realistic).

## How I would use it (practical)
- **Context:** Growing table size and slow queries.
- **Steps:** check dead tuples → tune autovacuum thresholds → run manual vacuum if needed.
- **What success looks like:** stable table size and consistent query latency.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** reclaims space and keeps indexes healthy.
- **Cons / Risks:** extra I/O and write overhead.
### Alternatives
- **Manual VACUUM FULL:** heavy, blocks writes.
- **How to choose:** prefer autovacuum; use VACUUM FULL only when necessary.

## Failure modes & Pitfalls
- Autovacuum too lax → bloat.
- Autovacuum too aggressive → write amplification.

## Observability (How to detect issues)
- **Metrics:** dead tuples, autovacuum lag, table size.
- **Logs:** autovacuum activity and duration.
- **Alerts:** rising dead tuples or bloat.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Monitor dead tuples
  - [ ] Tune autovacuum thresholds
  - [ ] Avoid long-running transactions

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Disabling autovacuum globally.
  - **Why it’s bad:** bloat and performance degradation.
  - **Better approach:** tune it per workload.

## Interview readiness
### “Explain it like I’m teaching”
- Vacuum in Postgres removes dead row versions created by MVCC. Autovacuum automates this; tuning it keeps performance stable and prevents bloat.

### Trap questions (with answers)
1) **Q:** Does VACUUM immediately shrink the file?
   - **A:** no; it marks space reusable but doesn’t always shrink files.
2) **Q:** Can you disable autovacuum safely?
   - **A:** generally no; bloat will accumulate.
3) **Q:** Is VACUUM FULL always better?
   - **A:** no; it’s blocking and heavy.

### Quick self-check (5 items)
- [ ] I can define vacuum and autovacuum.
- [ ] I can explain why bloat happens.
- [ ] I can name a tuning lever.
- [ ] I can describe a pitfall.
- [ ] I can explain VACUUM vs VACUUM FULL.

## Links (NO duplication)
### Prerequisites
- [PostgreSQL MVCC](postgresql-mvcc.md)

### Related topics
- [PostgreSQL bloat](postgresql-bloat.md)
- [Slow query diagnosis](slow-query-diagnosis.md)

### Compare with
- [Maintenance windows](../operations/maintenance-windows.md)
