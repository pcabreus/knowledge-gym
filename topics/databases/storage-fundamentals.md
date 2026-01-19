---
id: storage-fundamentals
title: "Storage Fundamentals"
type: concept
status: learning
importance: 40
difficulty: easy
tags: [database, storage]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Storage Fundamentals

## TL;DR (BLUF)
- Storage fundamentals cover I/O, latency, and space trade-offs.
- Use them to reason about database performance and maintenance.
- Trade-off: higher performance often costs more.

## Definition
**What it is:** Basic concepts of disk/SSD storage, I/O patterns, and latency.
**Key terms:** I/O, throughput, latency, fragmentation.

## Why it matters
- Storage behavior directly affects DB performance.
- Maintenance tasks like compaction and vacuum depend on storage.

## Scope & Non-goals
**In scope:** storage performance basics.
**Out of scope / NOT solved by this:** hardware procurement.

## Mental model / Intuition
- Storage is the slowest part of the stack; plan around it.

## Decision rules (When to use / When not to use)
### Use it when
- You diagnose latency spikes or bloat issues.
### Avoid it when
- Performance issues are clearly CPU-bound.

## How I would use it (practical)
- **Context:** Slow queries after data growth.
- **Steps:** check I/O metrics → review storage type → adjust maintenance.
- **What success looks like:** stable I/O and latency.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** better performance with faster storage.
- **Cons / Risks:** higher cost.
### Alternatives
- **Caching:** reduce storage I/O.
- **How to choose:** optimize I/O when storage is the bottleneck.

## Failure modes & Pitfalls
- Ignoring I/O saturation signals.

## Observability (How to detect issues)
- **Metrics:** disk I/O, queue depth, latency.
- **Logs:** storage errors.
- **Alerts:** sustained I/O saturation.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Monitor I/O and latency
  - [ ] Plan maintenance around I/O limits

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Running compaction during peak traffic.
  - **Why it’s bad:** I/O saturation.
  - **Better approach:** schedule maintenance windows.

## Interview readiness
### “Explain it like I’m teaching”
- Storage fundamentals explain why databases slow down: I/O latency and throughput are limiting factors. Understanding them helps plan indexes and maintenance.

### Trap questions (with answers)
1) **Q:** Is CPU always the bottleneck?
   - **A:** no; storage I/O often is.
2) **Q:** Does SSD remove the need for indexes?
   - **A:** no; indexes still reduce I/O.
3) **Q:** Can storage issues appear only at scale?
   - **A:** yes; I/O saturation is common at scale.

### Quick self-check (5 items)
- [ ] I can define storage fundamentals.
- [ ] I can name key metrics.
- [ ] I can describe a trade-off.
- [ ] I can explain a pitfall.
- [ ] I can link storage to DB performance.

## Links (NO duplication)
### Prerequisites
- [Performance basics](../system-design/performance-basics.md)

### Related topics
- [Storage compaction](storage-compaction.md)

### Compare with
- [Caching fundamentals](../system-design/caching-fundamentals.md) — memory vs storage.
