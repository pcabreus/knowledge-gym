---
id: sqlite
title: "SQLite"
type: technology
status: learning
importance: 50
difficulty: easy
tags: [database, sql, embedded]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# SQLite

## TL;DR (BLUF)
- Embedded SQL database stored in a single file.
- Use it for local apps, testing, and small to medium datasets.
- Trade-off: limited concurrency and scaling compared to client-server DBs.

## Definition
**What it is:** A lightweight, serverless relational database library that stores data in a file.
**Key terms:** embedded database, file-based, ACID.

## Why it matters
- It’s ubiquitous for local storage and testing.
- It provides SQL without operational overhead.

## Scope & Non-goals
**In scope:** local storage, low-ops SQL, single-node apps.
**Out of scope / NOT solved by this:** high-concurrency multi-node workloads.

## Mental model / Intuition
- Think of SQLite as a database you ship inside your app.

## Decision rules (When to use / When not to use)
### Use it when
- You need simple SQL with minimal ops.
- You have low to moderate concurrency.
### Avoid it when
- You need high write concurrency or horizontal scaling.
- You need advanced DB features from a server.

## How I would use it (practical)
- **Context:** Local desktop app or integration tests.
- **Steps:** define schema → use transactions → backup the file.
- **What success looks like:** reliable local persistence with minimal setup.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** zero-ops, fast for local use.
- **Cons / Risks:** limited concurrency and scaling.
### Alternatives
- **PostgreSQL:** for server-based multi-user workloads.
- **How to choose:** choose SQLite for embedded/local needs; switch to PostgreSQL when you need concurrency and scale.

## Failure modes & Pitfalls
- Write contention under concurrent workloads.
- File corruption without proper backups.

## Observability (How to detect issues)
- **Metrics:** write latency, lock contention.
- **Logs:** application logs for DB errors.
- **Alerts:** frequent write lock errors.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Use transactions
  - [ ] Backup the database file

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using SQLite for high-concurrency server workloads.
  - **Why it’s bad:** lock contention and latency spikes.
  - **Better approach:** use a client-server DB like [PostgreSQL](postgresql.md).

## Interview readiness
### “Explain it like I’m teaching”
- SQLite is an embedded SQL database in a single file. It’s great for local apps and tests, but it doesn’t handle high concurrency like server databases.

### Trap questions (with answers)
1) **Q:** Is SQLite a server you connect to?
   - **A:** no; it’s embedded in the application.
2) **Q:** Can SQLite handle huge multi-tenant workloads?
   - **A:** not well; concurrency and scaling are limited.
3) **Q:** Does SQLite support transactions?
   - **A:** yes; it supports ACID transactions.

### Quick self-check (5 items)
- [ ] I can define SQLite precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete use case.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [SQL foundations](sql-foundations.md)

### Related topics
- [PostgreSQL](postgresql.md)
### Compare with
- [PostgreSQL](postgresql.md) — client-server vs embedded.
