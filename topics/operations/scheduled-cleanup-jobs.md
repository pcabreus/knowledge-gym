---
id: scheduled-cleanup-jobs
title: "Scheduled Cleanup Jobs"
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

# Scheduled Cleanup Jobs

## TL;DR (BLUF)
- Scheduled jobs delete or archive expired data on a fixed cadence.
- Use them when TTL is too imprecise.
- Trade-off: operational overhead and job failures.

## Definition
**What it is:** A periodic process that deletes or archives expired records.
**Key terms:** cron, batch job, retention.

## Why it matters
- It enforces data retention and compliance rules.
- Poorly designed jobs can lock tables or spike I/O.

## Scope & Non-goals
**In scope:** batch cleanup patterns and trade-offs.
**Out of scope / NOT solved by this:** real-time deletion guarantees.

## Mental model / Intuition
- Think of it as a scheduled janitor.

## Decision rules (When to use / When not to use)
### Use it when
- You need precise deletion timing or compliance.
### Avoid it when
- Best-effort TTL is sufficient and cheaper.

## How I would use it (practical)
- **Context:** Deleting expired sessions nightly.
- **Steps:** schedule job → batch deletes → monitor duration.
- **What success looks like:** expired data removed with bounded load.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** precise control.
- **Cons / Risks:** operational overhead.
### Alternatives
- **TTL:** best-effort cleanup.
- **How to choose:** use jobs for strict timing; TTL for convenience.

## Failure modes & Pitfalls
- Long-running jobs locking tables.

## Observability (How to detect issues)
- **Metrics:** job duration, rows deleted.
- **Logs:** job failures.
- **Alerts:** missed schedules or timeouts.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Batch deletes
  - [ ] Run during off-peak hours

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Deleting millions of rows in one transaction.
  - **Why it’s bad:** locks and timeouts.
  - **Better approach:** batch in chunks.

## Interview readiness
### “Explain it like I’m teaching”
- Scheduled cleanup jobs are periodic tasks that delete expired data when TTL isn’t precise enough. They provide control but require operational care.

### Trap questions (with answers)
1) **Q:** Are scheduled jobs always better than TTL?
   - **A:** no; TTL is simpler when precision isn’t required.
2) **Q:** Can cleanup jobs run at peak traffic?
   - **A:** they can, but it risks contention.
3) **Q:** Do cleanup jobs guarantee immediate deletion?
   - **A:** no; they run on a schedule.

### Quick self-check (5 items)
- [ ] I can define scheduled cleanup jobs.
- [ ] I can state when to use them.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain a monitoring signal.

## Links (NO duplication)
### Prerequisites
- [TTL (Time-to-Live)](../databases/ttl.md)

### Related topics
- [DynamoDB TTL](../databases/dynamodb-ttl.md)

### Compare with
- [TTL (Time-to-Live)](../databases/ttl.md) — best-effort vs scheduled.
