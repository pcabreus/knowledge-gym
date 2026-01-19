---
id: postgresql-mvcc
title: "PostgreSQL MVCC"
type: concept
status: learning
importance: 60
difficulty: medium
tags: [database, postgresql, concurrency]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# PostgreSQL MVCC

## TL;DR (BLUF)
- MVCC lets readers and writers proceed without blocking by using row versions.
- Use it to understand isolation, vacuum, and bloat behavior.
- Trade-off: old row versions accumulate and must be vacuumed.

## Definition
**What it is:** Multi-Version Concurrency Control stores multiple row versions to provide snapshot reads.
**Key terms:** row versions, snapshots, visibility.

## Why it matters
- It explains why reads don’t block writes (and vice versa) in PostgreSQL.
- It also explains bloat and the need for vacuum.

## Scope & Non-goals
**In scope:** MVCC basics, visibility, impact on storage.
**Out of scope / NOT solved by this:** distributed concurrency across nodes.

## Mental model / Intuition
- Think of each update as writing a new version while old versions stay until vacuumed.

## Decision rules (When to use / When not to use)
### Use it when
- You need to reason about isolation and performance under concurrency.
### Avoid it when
- You’re trying to reason about locking alone (MVCC still matters).

## How I would use it (practical)
- **Context:** Investigating table bloat and slow updates.
- **Steps:** verify MVCC behavior → check dead tuples → tune autovacuum.
- **What success looks like:** stable table size and predictable query latency.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** high read/write concurrency.
- **Cons / Risks:** bloat and maintenance overhead.
### Alternatives
- **Lock-based concurrency only:** simpler but more blocking.
- **How to choose:** MVCC is core to Postgres; focus on tuning vacuum.

## Failure modes & Pitfalls
- Long-running transactions preventing cleanup.
- Misinterpreting visibility leading to “missing” updates.

## Observability (How to detect issues)
- **Metrics:** dead tuples, table size growth.
- **Logs:** autovacuum activity.
- **Alerts:** rising bloat or vacuum lag.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Monitor long-running transactions
  - [ ] Monitor dead tuples

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Allowing long transactions to run indefinitely.
  - **Why it’s bad:** vacuum can’t clean up old versions.
  - **Better approach:** keep transactions short.

## Interview readiness
### “Explain it like I’m teaching”
- MVCC in Postgres means each update creates a new row version. Readers see a snapshot without blocking writers, but old versions must be cleaned up by vacuum to avoid bloat.

### Trap questions (with answers)
1) **Q:** Does MVCC eliminate all locks?
   - **A:** no; locks still exist for certain operations.
2) **Q:** Are old row versions deleted immediately?
   - **A:** no; vacuum removes them later.
3) **Q:** Do long transactions affect MVCC cleanup?
   - **A:** yes; they prevent old versions from being removed.

### Quick self-check (5 items)
- [ ] I can define MVCC.
- [ ] I can explain why vacuum is needed.
- [ ] I can name a failure mode.
- [ ] I can describe a signal.
- [ ] I can relate MVCC to isolation.

## Links (NO duplication)
### Prerequisites
- [Transactions](transactions.md)

### Related topics
- [Locks](locks.md)
- [PostgreSQL bloat](postgresql-bloat.md)
- [Vacuum and autovacuum](postgresql-vacuum-autovacuum.md)

### Compare with
- [Lock-based concurrency control](lock-based-concurrency-control.md)
