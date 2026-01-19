---
id: redis-persistence
title: "Redis persistence (RDB/AOF)"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [redis, durability, storage]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Redis persistence (RDB/AOF)

## TL;DR (BLUF)
- Redis persistence controls how in-memory data is written to disk.
- RDB snapshots are compact but can lose recent writes; AOF is more durable but heavier.
- Trade-off: durability vs performance and operational overhead.

## Definition
**What it is:** The mechanisms Redis uses to persist data: RDB (periodic snapshots) and AOF (append-only log of writes).  
**Key terms:** RDB snapshot, AOF, fsync, rewrite, durability window.

## Why it matters
- It determines how much data you can lose on crash.
- Poor settings can cause latency spikes or long restarts.

## Scope & Non-goals
**In scope:** snapshot vs log persistence, recovery behavior, durability windows.  
**Out of scope / NOT solved by this:** full transactional durability or multi-node consistency guarantees.

## Mental model / Intuition
- RDB is a periodic photo; AOF is a diary of every change.

## Decision rules (When to use / When not to use)
### Use it when
- You want Redis data to survive restarts.
- You can tune durability vs performance based on workload.
### Avoid it when
- Redis is purely ephemeral and can be rebuilt quickly.
- You require zero data loss (Redis is not designed for that).

## How I would use it (practical)
- **Context:** Cache with partial durability for sessions.
- **Steps:** enable AOF with `everysec` → add periodic RDB snapshots → test recovery time.
- **What success looks like:** acceptable data loss window and stable latency.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** data survives restarts; flexibility in durability.
- **Cons / Risks:** extra disk I/O, slower writes, long rewrite pauses.
### Alternatives
- **Use Redis as cache only:** accept data loss.
- **Use a durable DB:** when durability requirements are strict.
- **How to choose:** if losing some recent writes is acceptable, tune AOF + RDB; otherwise keep Redis as a cache.

## Failure modes & Pitfalls
- AOF rewrite causing latency spikes or disk pressure.
- RDB snapshot intervals that allow too much data loss.
- Restart time dominated by huge AOF files.

## Observability (How to detect issues)
- **Metrics:** persistence latency, fsync duration, AOF size, last save time.
- **Logs:** background save errors, AOF rewrite failures.
- **Alerts:** repeated background save failures or long fsync times.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Pick AOF fsync policy (`always`, `everysec`, or `no`).
  - [ ] Configure RDB snapshot intervals for recovery goals.
  - [ ] Monitor AOF growth and rewrite success.
  - [ ] Test restart and recovery time regularly.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Enabling AOF `always` on latency-sensitive workloads.
  - **Why it’s bad:** synchronous fsync can spike latency.
  - **Better approach:** use `everysec` or cache-only mode.

## Interview readiness
### “Explain it like I’m teaching”
- Redis persistence can snapshot memory (RDB) or log every write (AOF). RDB is faster but loses more data; AOF is more durable but heavier.

### Trap questions (with answers)
1) **Q:** Does AOF guarantee zero data loss?
   - **A:** no; `everysec` can lose up to one second of data, and crashes can lose more.
2) **Q:** Is RDB enough for strict durability?
   - **A:** no; snapshots are periodic and can lose recent writes.
3) **Q:** Can persistence settings affect latency?
   - **A:** yes; fsync and rewrite operations can introduce spikes.

### Quick self-check (5 items)
- [ ] I can explain RDB vs AOF.
- [ ] I can choose a fsync policy.
- [ ] I can describe the data loss window.
- [ ] I can name a persistence failure mode.
- [ ] I can monitor persistence health.

## Links (NO duplication)
### Prerequisites
- [Redis](redis.md)
- [Storage fundamentals](storage-fundamentals.md)

### Related topics
- [Redis eviction policies](redis-eviction-policies.md)
- [Redis scaling (scale up and out)](redis-scaling.md)

### Compare with
- [PostgreSQL](postgresql.md) — full durable RDBMS vs optional Redis persistence.

## Notes / Inbox (optional)
- N/A
