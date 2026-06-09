---
id: pgbouncer
title: "PgBouncer"
type: technology
status: learning
importance: 60
difficulty: medium
tags: [postgresql, database, connection-pooling, performance, operations]
canonical: true
aliases: []
created_at: 2026-06-09
updated_at: 2026-06-09
---

# PgBouncer

## TL;DR (BLUF)
- A lightweight, external connection pooler for PostgreSQL that multiplexes many client connections onto few server connections.
- Use it when many app instances/serverless functions exhaust PostgreSQL's `max_connections`.
- Trade-off: transaction pooling (the high-value mode) breaks session-scoped features (prepared statements, `SET`, advisory locks, `LISTEN/NOTIFY`).

## Definition
**What it is:** A standalone proxy that sits in front of PostgreSQL and reuses a small set of server connections across a large number of client connections.
**Key terms:** pooling mode (session/transaction/statement), `pool_size`, `max_client_conn`, `reserve_pool`, server vs client connection.

## Why it matters
- PostgreSQL forks a backend process per connection, so each connection costs RAM and scheduling overhead; a few hundred connections can degrade the whole cluster.
- Modern fan-out (many pods, serverless, autoscaling) easily exceeds `max_connections`, causing `too many clients` errors. PgBouncer absorbs that fan-out cheaply.

## Scope & Non-goals
**In scope:** connection multiplexing, pooling modes, and protecting PostgreSQL from connection storms.
**Out of scope / NOT solved by this:** query optimization, read/write splitting, sharding, and failover (PgBouncer is not a load balancer or HA layer by itself).

## Mental model / Intuition
- Think of PgBouncer as a turnstile: thousands of clients queue at the door, but only `pool_size` get a seat (a real PostgreSQL backend) at a time.
- In transaction mode the seat is released the instant a transaction commits, so seats turn over very fast.

## Decision rules (When to use / When not to use)
### Use it when
- App-side [connection pooling](connection-pooling.md) alone can't cap total connections (many instances × per-instance pools).
- Workloads are short transactions and you can tolerate transaction-level pooling semantics.
### Avoid it when
- You rely on session state (prepared statements without protocol support, session-level `SET`, advisory locks, `LISTEN/NOTIFY`) and can't switch to session pooling.
- A single well-sized client pool already keeps connections bounded.

## How I would use it (practical)
- **Context:** Kubernetes service with many replicas hammering one PostgreSQL primary.
- **Steps:** deploy PgBouncer (sidecar or central) → set `pool_mode = transaction` → set `pool_size` to match PostgreSQL capacity (not the number of clients) → shrink each app's local pool → monitor `SHOW POOLS`.
- **What success looks like:** server-side connections stay flat and well below `max_connections` while client concurrency grows.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Drastically reduces PostgreSQL backend count and memory pressure.
  - Cheap, single-purpose, battle-tested; transaction mode gives huge fan-in.
- **Cons / Risks:**
  - Transaction/statement modes break session-scoped features.
  - Another hop and another component to operate and monitor; can become a bottleneck if undersized.
  - Historically single-process/single-threaded (mitigated by `so_reuseport` + multiple instances in recent versions).
### Alternatives
- **Pgpool-II:** when you also want read/write splitting and load balancing (heavier).
- **Odyssey:** multithreaded pooler for very high connection counts.
- **Managed proxies (RDS Proxy, Supavisor):** when you want pooling without operating it.
- **How to choose:** PgBouncer for lean transaction pooling; Pgpool-II/managed proxies when you need routing/HA too. See [Database proxies](../operations/database-proxies.md).

## Failure modes & Pitfalls
- Using transaction mode with a driver that issues server-side prepared statements → "prepared statement does not exist" errors.
- `pool_size` set to the client count → no fan-in benefit; PostgreSQL still overloaded.
- `max_client_conn` too low → clients rejected at the pooler instead of queued.
- Long-running transactions hold a server connection and starve the pool (head-of-line blocking).

## Observability (How to detect issues)
- **Metrics:** `SHOW POOLS` (cl_active, cl_waiting, sv_active, sv_idle), `SHOW STATS` (avg query time, wait time), server connection count on PostgreSQL.
- **Logs:** "no more connections allowed", auth failures, query timeouts.
- **Traces:** added latency at the pooler hop.
- **Alerts:** rising `cl_waiting` (clients queued for a server connection) or saturated `pool_size`.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Pick pooling mode deliberately (transaction by default; session only if needed)
  - [ ] Size `pool_size` to PostgreSQL capacity, not client count
  - [ ] Shrink app-side pools after introducing PgBouncer
  - [ ] Configure auth (`auth_type`, `auth_query`) and TLS
  - [ ] Monitor `cl_waiting` and server connection count
- **Operational notes**
  - Run multiple PgBouncer instances behind `so_reuseport` to use more than one core.

## Mini example (if applicable)
```ini
# pgbouncer.ini
[databases]
appdb = host=10.0.0.5 port=5432 dbname=appdb

[pgbouncer]
listen_port = 6432
auth_type = scram-sha-256
pool_mode = transaction
max_client_conn = 5000
default_pool_size = 20      # only 20 real PostgreSQL backends per user/db
```

## Common anti-patterns
- **Anti-pattern:** Enabling transaction pooling while the driver uses server-side prepared statements.
  - **Why it’s bad:** statements bound to one server connection vanish when the connection is reassigned.
  - **Better approach:** disable server-side prepared statements, or use session pooling.
- **Anti-pattern:** Adding PgBouncer but leaving large client pools.
  - **Why it’s bad:** total connections still explode.
  - **Better approach:** reduce client pools and let PgBouncer do the multiplexing.

## Interview readiness
### “Explain it like I’m teaching”
- PgBouncer is a thin proxy in front of PostgreSQL that lets thousands of clients share a handful of real database connections. In transaction mode it hands a server connection to a client only for the duration of a transaction, so a small pool serves huge concurrency — at the cost of losing session-scoped features.

### Trap questions (with answers)
1) **Q:** Does PgBouncer replace app-side connection pooling?
   - **A:** No — you still want sane client pools; PgBouncer caps the *total* server connections across all clients.
2) **Q:** Why can transaction pooling break prepared statements?
   - **A:** A server connection is reassigned between transactions, so state bound to a specific backend (prepared statements, `SET`, advisory locks) doesn't carry over.
3) **Q:** If clients are queueing (`cl_waiting` high) but PostgreSQL CPU is low, what's likely wrong?
   - **A:** `pool_size` is too small (or long transactions hold connections) — the bottleneck is the pool, not the database.
4) **Q:** Is PgBouncer a load balancer or HA solution?
   - **A:** No — it only multiplexes connections; routing/replicas/failover need other tooling (e.g. Pgpool-II, managed proxies).

### Quick self-check (5 items)
- [ ] I can define what PgBouncer does in 2–3 sentences.
- [ ] I can explain the three pooling modes and their trade-offs.
- [ ] I can explain why `pool_size` should track DB capacity, not client count.
- [ ] I can name a session feature transaction pooling breaks.
- [ ] I can name a monitoring signal (`cl_waiting`).

## Links (NO duplication)
### Prerequisites
- [Connection pooling](connection-pooling.md)
- [Database client basics](database-client-basics.md)
- [PostgreSQL](postgresql.md)

### Related topics
- [PostgreSQL scaling playbook](postgresql-scaling-playbook.md)
- [Scaling Reads](../system-design/scaling-reads.md)

### Compare with
- [Database proxies](../operations/database-proxies.md) — PgBouncer is a specific, PostgreSQL-only connection pooler within that broader category.
