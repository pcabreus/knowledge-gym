---
id: database-proxies
title: "Database Proxies"
type: technology
status: learning
importance: 45
difficulty: medium
tags: [operations, database]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Database Proxies

## TL;DR (BLUF)
- DB proxies manage connections and pooling between apps and databases.
- Use them to stabilize connections and reduce database load.
- Trade-off: added latency and another component to operate.

## Definition
**What it is:** A middle layer that multiplexes client connections to a DB.
**Key terms:** pooling, multiplexing, connection limits.

## Why it matters
- It prevents connection storms and improves stability.
- It can add latency and operational complexity.

## Scope & Non-goals
**In scope:** connection management and pooling.
**Out of scope / NOT solved by this:** query optimization.

## Mental model / Intuition
- Think of a proxy as a traffic controller for DB connections.

## Decision rules (When to use / When not to use)
### Use it when
- Many app instances create too many connections.
### Avoid it when
- You already have stable pool sizes and low connection churn.

## How I would use it (practical)
- **Context:** Serverless workloads with connection spikes.
- **Steps:** add proxy → reduce client pool sizes → monitor latency.
- **What success looks like:** stable connections and fewer timeouts.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** connection stability and pooling.
- **Cons / Risks:** extra hop and operational overhead.
### Alternatives
- **Client pooling only:** simpler if stable.
- **How to choose:** use proxies when connection storms are a risk.

## Failure modes & Pitfalls
- Proxy saturation causing cascading timeouts.

## Observability (How to detect issues)
- **Metrics:** connection counts, proxy latency.
- **Logs:** proxy errors.
- **Alerts:** spike in proxy latency or errors.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Tune pool sizes on clients
  - [ ] Monitor proxy latency

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Adding a proxy without adjusting client pools.
  - **Why it’s bad:** still overloads DB.
  - **Better approach:** reduce client pools after proxying.

## Interview readiness
### “Explain it like I’m teaching”
- Database proxies sit between apps and DBs to manage connections and pooling. They reduce connection storms but add another operational layer.

### Trap questions (with answers)
1) **Q:** Do proxies remove the need for pooling?
   - **A:** no; clients still need sane pool sizes.
2) **Q:** Are proxies always faster?
   - **A:** no; they add a hop.
3) **Q:** Can proxies fix slow queries?
   - **A:** no; they don’t optimize queries.

### Quick self-check (5 items)
- [ ] I can define DB proxies.
- [ ] I can state when to use them.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain a monitoring signal.

## Links (NO duplication)
### Prerequisites
- [Database client basics](../databases/database-client-basics.md)

### Related topics
- [Connection pooling](../databases/connection-pooling.md)

### Compare with
- [Connection pooling](../databases/connection-pooling.md) — proxy pooling vs client pooling.
