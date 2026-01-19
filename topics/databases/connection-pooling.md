---
id: connection-pooling
title: "Connection Pooling"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [database, performance]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Connection Pooling

## TL;DR (BLUF)
- A pool reuses DB connections to reduce connection overhead.
- Use it to stabilize latency and limit DB load.
- Trade-off: too many connections can overwhelm the DB.

## Definition
**What it is:** A managed set of reusable database connections.
**Key terms:** pool size, max connections, idle timeout.

## Why it matters
- Creating connections is expensive and can throttle the DB.
- Poor pool sizing causes latency spikes or overload.

## Scope & Non-goals
**In scope:** pool sizing, queueing, and operational impact.
**Out of scope / NOT solved by this:** client timeout/retry configuration.

## Mental model / Intuition
- Think of a shared set of reusable sockets to the DB.

## Decision rules (When to use / When not to use)
### Use it when
- You have frequent DB access from many requests.
### Avoid it when
- You already saturate the DB with too many connections.

## How I would use it (practical)
- **Context:** API service with high request rate.
- **Steps:** set pool size based on DB capacity → set timeouts → monitor queueing.
- **What success looks like:** stable latency and no connection storms.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** lower latency and controlled DB load.
- **Cons / Risks:** misconfigured pools can cause queueing or overload.
### Alternatives
- **Serverless DB proxies:** managed pooling.
- **How to choose:** use pooling by default and tune to DB limits.

## Failure modes & Pitfalls
- Pool too large causing DB overload.
- Pool too small causing request queueing.

## Observability (How to detect issues)
- **Metrics:** active connections, pool wait time, query latency.
- **Logs:** connection timeout errors.
- **Alerts:** rising pool wait time or DB connection saturation.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Set max connections per service
  - [ ] Configure idle timeout
  - [ ] Monitor pool wait time

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Each request opens a new DB connection.
  - **Why it’s bad:** connection storms and latency spikes.
  - **Better approach:** use a pool.

## Interview readiness
### “Explain it like I’m teaching”
- Connection pooling reuses a fixed number of DB connections to reduce overhead and protect the database. It improves latency but must be sized carefully.

### Trap questions (with answers)
1) **Q:** Is a bigger pool always better?
   - **A:** no; too many connections can overwhelm the DB.
2) **Q:** Can pooling hide slow queries?
   - **A:** no; it can actually increase queueing.
3) **Q:** Do you need pooling for serverless?
   - **A:** often yes, or use a managed proxy.

### Quick self-check (5 items)
- [ ] I can define connection pooling.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can explain a failure mode.
- [ ] I can describe a monitoring signal.

## Links (NO duplication)
### Prerequisites
- [Database client basics](database-client-basics.md)

### Related topics
- [Slow query diagnosis](slow-query-diagnosis.md)

### Compare with
- [Database proxies](../operations/database-proxies.md)
