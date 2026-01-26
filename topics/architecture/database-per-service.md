---
id: database-per-service
title: "Database per Service"
type: pattern
status: learning
importance: 75
difficulty: hard
tags: [architecture, microservices, data-management, decoupling]
canonical: true
aliases: [private database, service-owned data]
created_at: 2026-01-26
updated_at: 2026-01-26
---

# Database per Service

## TL;DR (BLUF)
- Each microservice owns its database/schema exclusively; no shared databases.
- Use it to achieve true service autonomy and independent deployments.
- Trade-off: distributed data consistency, cross-service queries, and data duplication.

## Definition
**What it is:** An architectural pattern where each microservice has exclusive ownership of its database (or schema), and other services can only access that data via the service's API, never directly.

**Key terms:** service autonomy, data ownership, bounded context, polyglot persistence, no shared database, eventual consistency.

## Why it matters
- Enables true service independence: deploy, scale, and evolve services separately.
- Prevents tight coupling via shared database (common microservices anti-pattern).
- Allows polyglot persistence: each service chooses optimal database (Postgres, Mongo, Redis).
- Enforces bounded context boundaries ([Domain-Driven Design](domain-driven-design.md)).

## Scope & Non-goals
**In scope:** service data ownership, API-based data access, eventual consistency, data synchronization patterns.

**Out of scope / NOT solved by this:**
- Strong consistency across services (use [ACID](../databases/acid-properties.md) within a service)
- Cross-service joins (use [API Composition](api-composition.md) or [CQRS](cqrs.md))
- Distributed transactions (use [Saga](saga-pattern.md))

## Mental model / Intuition
- Like separate companies: each has its own accounting system; they exchange invoices via contracts, not direct DB access.
- In microservices: `user-service` owns `users` table; `order-service` cannot query it directly—must call `/users` API.

## Decision rules (When to use / When not to use)
### Use it when
- Building microservices with true autonomy (deploy independently).
- Services have distinct bounded contexts (users, orders, payments).
- Teams own end-to-end responsibility (dev, deploy, DB schema).
- You need polyglot persistence (mix Postgres, Mongo, DynamoDB).

### Avoid it when
- Building a modular monolith (shared DB is fine).
- Strong consistency is critical across all data (banking, inventory).
- Team lacks distributed systems expertise.
- Operational overhead of managing N databases is too high.

## How I would use it (practical)
- **Context:** E-commerce platform with `user-service`, `order-service`, and `payment-service`.
- **Steps:**
  1) **Assign database ownership:**
     - `user-service` → `users_db` (Postgres)
     - `order-service` → `orders_db` (Postgres)
     - `payment-service` → `payments_db` (DynamoDB)
  2) **Enforce API-only access:**
     - `order-service` needs user info → calls `GET /users/{id}` API (not direct DB query).
     - Database credentials for `users_db` are **private** to `user-service`.
  3) **Handle data duplication:**
     - `order-service` caches `userName` locally (eventual consistency).
     - When user updates name, `user-service` publishes `UserUpdated` event.
     - `order-service` subscribes and updates local cache.
  4) **Use Saga for distributed transactions:**
     - Checkout flow: `order-service` → `payment-service` → `inventory-service`.
     - Use [Saga pattern](saga-pattern.md) with compensations.
  5) **Monitor data consistency:**
     - Track sync lag between services.
     - Alert on stale data (if `order.userName` ≠ `user.name` for >1 hour).

## Trade-offs / Costs (and their mitigation)
| Trade-off | Mitigation |
|-----------|-----------|
| No cross-service joins (SQL) | Use [API Composition](api-composition.md) or [CQRS](cqrs.md) read models |
| Data duplication (eventual consistency) | Accept staleness; use events for sync; monitor lag |
| Distributed transaction complexity | Use [Saga pattern](saga-pattern.md); avoid when possible |
| Operational overhead (N databases) | Use managed databases (AWS RDS, Cloud SQL); automate provisioning |
| Data migration harder | Plan schema changes per service; use versioning |

## Failure modes / Edge cases
1. **Service calls another service's DB directly:** Violates encapsulation.
   - *Mitigation:* Enforce network policies (DB firewall); code reviews.
2. **Cross-service query explosion:** "Order details" needs 10 API calls.
   - *Mitigation:* Use [BFF](backend-for-frontend.md) or [CQRS](cqrs.md) read model.
3. **Data consistency drift:** `order.userName` stale after user update.
   - *Mitigation:* Use [Outbox pattern](outbox-pattern.md) for reliable events; monitor staleness.
4. **Schema migration deadlock:** Service A depends on Service B's schema.
   - *Mitigation:* Use [API versioning](../system-design/versioning-apis-and-events.md); backward compatibility.
5. **Reporting/analytics nightmare:** Need to join data from 10 services.
   - *Mitigation:* Use [Change Data Capture](../databases/change-data-capture.md) to replicate to data warehouse.

## Alternatives
- **Shared database (monolith):** Simple but couples services tightly.
- **Shared schema per bounded context:** Multiple services share DB but different schemas (hybrid).
- **Database-as-a-Service layer:** Central data service owns DB, others call it (centralized).
- **Event sourcing + CQRS:** Store events centrally, build per-service read models.

## Data access patterns
### 1. API calls (synchronous)
- Service A calls Service B's REST/gRPC API.
- **Pros:** Simple, strongly typed.
- **Cons:** Latency, cascading failures.

### 2. Event-driven (asynchronous)
- Service A publishes event; Service B subscribes and updates local cache.
- **Pros:** Decoupled, resilient.
- **Cons:** Eventual consistency, complexity.

### 3. CQRS read model
- Dedicated read database aggregates data from multiple services.
- **Pros:** Fast queries, no runtime joins.
- **Cons:** Eventual consistency, dual-write issues.

### 4. Data warehouse / ETL
- Batch replicate data to warehouse for analytics.
- **Pros:** Optimized for reporting.
- **Cons:** Not for operational queries; staleness.

**Recommendation:** Use **API calls** for real-time, **events** for async sync, **CQRS** for complex queries, **warehouse** for analytics.

## Combinations
Database per Service is **almost always used with:**
- **[Saga pattern](saga-pattern.md):** Distributed transactions across services.
- **[Outbox pattern](outbox-pattern.md):** Reliable event publishing.
- **[CQRS](cqrs.md):** Separate read models for queries.
- **[API Composition](api-composition.md):** Aggregate data at runtime.
- **[Bounded Contexts](bounded-contexts.md):** DDD foundation for service boundaries.
- **[Observability](../operations/observability-basics.md):** Track data consistency, sync lag.

**Typical combination:**
- **Microservices architecture:** Database per Service + Saga + Outbox + CQRS (if needed) + Observability

## Prerequisites
- Understanding of [Microservices design](microservices-design-basics.md).
- Familiarity with [Domain-Driven Design](domain-driven-design.md) and [Bounded Contexts](bounded-contexts.md).
- Knowledge of [Event-driven architecture](event-driven-basics.md) and [Saga](saga-pattern.md).

## Related topics
- [Saga pattern](saga-pattern.md): Distributed transactions.
- [Outbox pattern](outbox-pattern.md): Reliable event publishing.
- [CQRS](cqrs.md): Separate read models.
- [API Composition](api-composition.md): Aggregate data via APIs.
- [Bounded Contexts](bounded-contexts.md): Define service boundaries.
- [Change Data Capture](../databases/change-data-capture.md): Replicate data for analytics.

## Real-world examples
1. **Netflix:** Each microservice owns its Cassandra keyspace or database.
2. **Amazon:** Services use different databases (DynamoDB, Aurora, S3) based on needs.
3. **Uber:** Separate databases for riders, drivers, trips, payments.

## Checklist (self-test)
- [ ] I understand the difference between database per service and shared database.
- [ ] I can design API contracts for cross-service data access.
- [ ] I know how to handle data duplication and eventual consistency.
- [ ] I can explain when to use Saga vs CQRS for cross-service queries.
- [ ] I can monitor data consistency and sync lag.

## Reminders / Key takeaways
- **One service, one database** (or schema).
- **No direct DB access** across services (enforce with network policies).
- Accept **eventual consistency** for cross-service data.
- Use [Saga](saga-pattern.md) for distributed transactions.
- Use [CQRS](cqrs.md) for complex cross-service queries.

## Trap questions (with answers)
### Q1: Can two services share the same database instance but different schemas?
**A:** **Technically yes, but risky**. Sharing a DB instance (e.g., Postgres server) is fine for cost savings, but each service must have its **own schema** with **no cross-schema queries**. Better: use separate databases entirely to enforce isolation and allow polyglot persistence. Shared instance can still create coupling (shared resource limits, schema migration conflicts).

### Q2: How do I query data from multiple services (e.g., order + user + payment)?
**A:** **Options:**
1. **[API Composition](api-composition.md):** Call multiple services, aggregate in [BFF](backend-for-frontend.md) (runtime overhead).
2. **[CQRS](cqrs.md):** Build read-only view that subscribes to events from all services (eventual consistency).
3. **Data warehouse:** ETL data to warehouse for reporting (not for operational queries).
Choose based on latency, consistency, and query complexity. For UI, use API Composition or CQRS.

### Q3: What if I need strong consistency across services?
**A:** **You can't** (distributed transactions are anti-pattern in microservices). Instead:
1. **Keep strongly consistent data in one service** (e.g., order + order-items in order-service).
2. Use [Saga pattern](saga-pattern.md) for **eventual consistency** with compensations.
3. Reconsider service boundaries: maybe data belongs together ([Bounded Contexts](bounded-contexts.md)).
If you truly need ACID across services, you may need a monolith or modular monolith.

### Q4: Can I use a shared read-only database for reporting?
**A:** **Yes, but not directly from services' databases**. Instead:
1. Use [Change Data Capture](../databases/change-data-capture.md) to replicate data to **data warehouse**.
2. Analytics queries run on warehouse, not production DBs.
3. Avoid: services directly querying each other's DBs (breaks encapsulation).
Reporting is a valid use case for data replication, not for breaking database-per-service rule.

### Q5: How do I migrate from shared database to database per service?
**A:** Use **[Strangler Fig](strangler-fig.md)**:
1. Identify bounded contexts (future services).
2. Extract one service at a time.
3. Migrate data incrementally (dual-write, CDC, batch migration).
4. Refactor cross-service queries to API calls or events.
5. Eventually decommission shared database.
Do NOT attempt "one big split"—too risky. Incremental migration with clear rollback plan. See [Strangler Fig](strangler-fig.md).
