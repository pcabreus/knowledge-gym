---
id: maintenance-windows
title: "Maintenance Windows"
type: pattern
status: learning
importance: 45
difficulty: easy
tags: [operations, maintenance]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Maintenance Windows

## TL;DR (BLUF)
- Maintenance windows are planned periods for heavy maintenance tasks.
- Use them for operations that impact performance (vacuum full, compaction).
- Trade-off: reduced availability during the window.

## Definition
**What it is:** A scheduled time window for disruptive maintenance tasks.
**Key terms:** maintenance, downtime, off-peak.

## Why it matters
- It reduces user impact from heavy operations.
- It requires coordination and communication.

## Scope & Non-goals
**In scope:** planning maintenance tasks.
**Out of scope / NOT solved by this:** automated zero-downtime upgrades.

## Mental model / Intuition
- Think of it as a planned service pause.

## Decision rules (When to use / When not to use)
### Use it when
- Maintenance tasks are heavy and disruptive.
### Avoid it when
- You can do online maintenance without impact.

## How I would use it (practical)
- **Context:** VACUUM FULL on large tables.
- **Steps:** schedule window → notify stakeholders → run tasks.
- **What success looks like:** maintenance completed with limited impact.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** controlled risk.
- **Cons / Risks:** downtime or degraded performance.
### Alternatives
- **Online maintenance:** less disruption.
- **How to choose:** use windows when online options aren’t safe.

## Failure modes & Pitfalls
- Maintenance overruns causing extended downtime.

## Observability (How to detect issues)
- **Metrics:** maintenance duration, error rate.
- **Logs:** task failures.
- **Alerts:** maintenance overruns.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Announce window
  - [ ] Run heavy tasks
  - [ ] Validate service health

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Running heavy maintenance during peak traffic.
  - **Why it’s bad:** user impact.
  - **Better approach:** schedule off-peak windows.

## Interview readiness
### “Explain it like I’m teaching”
- Maintenance windows are planned times to run heavy operations with minimal user impact. They’re a trade-off between availability and safety.

### Trap questions (with answers)
1) **Q:** Are maintenance windows always necessary?
   - **A:** no; some operations can be online.
2) **Q:** Should you skip communication for short windows?
   - **A:** no; stakeholders should be informed.
3) **Q:** Is maintenance the same as backups?
   - **A:** no; backups are separate operational tasks.

### Quick self-check (5 items)
- [ ] I can define maintenance windows.
- [ ] I can state when to use them.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain a monitoring signal.

## Links (NO duplication)
### Prerequisites
- [Operational planning](operational-planning.md)

### Related topics
- [PostgreSQL vacuum & autovacuum](../databases/postgresql-vacuum-autovacuum.md)

### Compare with
- [Storage compaction](../databases/storage-compaction.md) — heavy maintenance vs routine cleanup.
