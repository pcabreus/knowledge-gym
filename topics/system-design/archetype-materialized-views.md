---
id: archetype-materialized-views
title: "Materialized Views / CQRS Read Models"
type: pattern
status: learning
importance: 90
difficulty: hard
tags: [system-design, cqrs, materialized-views, read-projection, eventual-consistency]
canonical: true
aliases: [cqrs-read-models, read-projection, precomputed-view, serving-layer]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Materialized Views / CQRS Read Models

## TL;DR (BLUF)
- Maintain one or more read-optimized representations derived from write events / source of truth.
- Key challenge: staleness (eventual consistency), rebuild complexity, dual-write bugs.
- Solution patterns: CQRS + event-driven projections, outbox pattern, replayable log.

## Definition
**What it is:** A system archetype that maintains specialized read-optimized data structures (projections/views) derived from the authoritative write model or event stream. Separates read and write responsibilities.

**Key terms:**
- **Source of truth**: The authoritative write model (often event log or transactional DB)
- **Projection / Read model**: Derived view optimized for specific queries
- **Eventual consistency**: Projections lag behind source; eventually consistent
- **Rebuild**: Recreating a projection from the source (replay events)

## Why it matters
- **Query performance**: Complex read queries on write-optimized models are slow; projections are pre-computed
- **Read scalability**: Multiple read models can scale independently
- **Separation of concerns**: Different read use cases don't conflict
- **Interview frequency**: Common in event-driven, CQRS system design

**Typical real-world consequences of getting it wrong:**
- Marketplace: Seller dashboard metrics computed on-demand → query timeout, p99 latency spikes
- Search: No search index → full-table scans, terrible UX
- Analytics: Ad-hoc queries on OLTP DB → locks, production outages

## Scope & Non-goals
**In scope:**
- Building and maintaining read projections
- Handling staleness and consistency expectations
- Rebuild strategies when schema changes
- Preventing dual-write bugs

**Out of scope / NOT solved by this:**
- Event ingestion → see [High-Throughput Event Ingestion](archetype-event-ingestion.md)
- Write model design → see [Event Sourcing](../architecture/event-sourcing.md)
- Multi-region consistency → see [Multi-Region Replication](archetype-multi-region.md)

## Mental model / Intuition
Think of it like **library catalog systems**:
- Source of truth: The books themselves (actual inventory)
- Projections: Card catalog (by title), author index, subject index
- Each index is a different view optimized for specific searches
- When a book is added, catalogs are updated (eventually)
- Rebuilding: If a catalog is lost, recreate it from the books

## Decision rules (When to use / When not to use)
### Use it when
- **Read queries don't match write model**: Need joins, aggregations, denormalization
- **Read scalability needed**: Writes are low but reads are high
- **Multiple read patterns**: Dashboard, search, reports need different shapes
- **Event-driven architecture**: Already producing events; projections are natural

### Avoid it when
- **Simple CRUD**: Reads match writes 1:1; no complex queries
- **Strong consistency required**: Can't tolerate staleness
- **Low scale**: Extra complexity not justified

## How I would use it (practical)
### Context: Marketplace seller dashboard
**Problem:** Sellers need metrics: revenue/day, refund rate, top products. Orders are in normalized relational DB. Computing on the fly causes slow queries and locks.

**Steps:**
1. **Source of truth**: Orders table + events (`OrderPaid`, `OrderRefunded`)
2. **Projection**: `seller_dashboard` table with columns `seller_id`, `date`, `revenue`, `refund_count`, `top_products_json`
3. **Event-driven update**:
   - Listen to `OrderPaid` → increment revenue, update top products
   - Listen to `OrderRefunded` → decrement revenue, increment refund_count
4. **Outbox pattern**: Emit events reliably from Orders DB to ensure no missed updates
5. **Rebuild strategy**: Reprocess all historical orders when adding new metrics
6. **Consistency**: Dashboard shows "(as of 10s ago)" to set expectations

**What success looks like:**
- Dashboard p99 < 200ms (vs 5s before)
- Staleness < 30s p99
- No production locks from dashboard queries

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Fast reads (pre-computed)
  - Read/write scaling independence
  - Multiple views without impacting writes
- **Cons / Risks:**
  - Staleness (eventual consistency)
  - Rebuild complexity when schema changes
  - Dual-write bugs if not event-driven
  - Operational overhead (more storage, more services)

### Alternatives
- **Query on write model directly:** Simple but slow at scale
  - **When better:** Low traffic, simple queries, strong consistency needed
- **Denormalize in write model:** Faster reads but write complexity and data duplication
  - **When better:** Fixed read patterns, small scale
- **Cache results:** Faster but still computes on miss; doesn't solve all query shapes
  - **When better:** Reads are identical and cacheable

**How to choose:**
- Start with direct queries; add caching for hot queries
- If caching insufficient or query diversity high → materialized views
- Combine caching + materialized views for best performance

## Failure modes & Pitfalls
### Where it hurts (why it hurts)
1. **Staleness (eventual consistency) → users see old data**
   - **Symptom:** User makes a change; dashboard doesn't reflect it for seconds/minutes
   - **Why:** Projection update lags behind write
   - **Solution:** Set user expectations ("updated 10s ago"); critical reads go to source; monitor lag

2. **Rebuild complexity when schema changes**
   - **Symptom:** Adding new metric requires replaying all historical events; takes hours/days
   - **Why:** Projection doesn't store intermediate state; must recompute from source
   - **Solution:** Store events in replayable log; design projections to be rebuildable; blue-green deployment

3. **Dual-write bugs if not event-driven**
   - **Symptom:** Write to source succeeds but projection update fails → inconsistent state
   - **Why:** Separate write operations without transactional guarantee
   - **Solution:** Always use [Outbox Pattern](../architecture/outbox-pattern.md) or event sourcing

## Observability (How to detect issues)
### Metrics
- `projection_lag_seconds`: Time behind source (critical SLI)
- `projection_rebuild_duration`: Time to rebuild from scratch
- `projection_update_errors`: Failed event processing

### Logs
- Log each projection update: event_id, timestamp, outcome

### Traces
- Span: `write_source` → `emit_event` → `update_projection`

### Alerts
- `projection_lag > 60s for 5 minutes` → projection falling behind
- `projection_update_errors > 1%` → processing bugs

## Implementation notes
### Checklist
- [ ] Identify read patterns that don't match write model
- [ ] Design projection schema (denormalized for read efficiency)
- [ ] Set up event stream from source (use [Outbox Pattern](../architecture/outbox-pattern.md))
- [ ] Implement idempotent projection updaters (handle duplicate events)
- [ ] Plan rebuild strategy (replay historical events)
- [ ] Monitor projection lag
- [ ] Document staleness expectations for users

### Performance notes
- **Projection updates**: Should be idempotent (events may be redelivered)
- **Rebuild**: Parallelize by partition key for faster reprocessing

### Operational notes
- **Blue-green rebuild**: Build new projection version alongside old; cutover when caught up
- **Tombstone events**: Support deletion/correction events to handle mistakes

## Mini example (conceptual)
```python
# Source: OrderService writes to DB + emits events via Outbox
class OrderService:
    def mark_paid(self, order_id, amount):
        with transaction:
            order = db.orders.update(order_id, status='paid')
            outbox.publish('OrderPaid', {
                'order_id': order_id,
                'seller_id': order.seller_id,
                'amount': amount,
                'timestamp': now()
            })

# Projection: DashboardProjection listens to events
class DashboardProjection:
    def on_order_paid(self, event):
        # Idempotent: use event_id to deduplicate
        if processed(event.id):
            return
        
        db.seller_dashboard.upsert(
            seller_id=event.seller_id,
            date=event.timestamp.date(),
            increment={'revenue': event.amount, 'order_count': 1}
        )
        mark_processed(event.id)

# Rebuild: Replay all historical events
def rebuild_dashboard():
    for event in event_store.replay('OrderPaid', from_beginning=True):
        projection.on_order_paid(event)
```

## Common anti-patterns
### Anti-pattern: Dual-write (update source + projection in separate calls)
- **What people do:** Write to DB, then call projection service to update
- **Why it's bad:** Projection call can fail; no rollback → inconsistent state
- **Better approach:** Use [Outbox Pattern](../architecture/outbox-pattern.md) for atomic write + event emission

### Anti-pattern: No rebuild strategy
- **What people do:** Build projection once; if corrupted or schema changes, manual recovery
- **Why it's bad:** Schema evolution and bug fixes become impossible
- **Better approach:** Design for rebuild from day one; keep replayable event log

### Anti-pattern: Ignoring staleness in UX
- **What people do:** Show projection data as if real-time without disclaimers
- **Why it's bad:** Users report "bugs" when they don't see immediate updates
- **Better approach:** Display "as of X seconds ago" or refresh timestamp; critical flows read from source

### Anti-pattern: Too many projections
- **What people do:** Create a projection for every slight variation in query
- **Why it's bad:** Operational burden; projection lag; storage explosion
- **Better approach:** Design projections to support multiple related queries; use filters/pagination on read

## Interview readiness
### "Explain it like I'm teaching"
Materialized views or CQRS read models are pre-computed, read-optimized representations of your data. The source of truth is the write model (often an event log or transactional DB). We derive one or more projections—like a dashboard view or search index—by processing events. This separates reads from writes: reads are fast because data is pre-shaped, and writes don't slow down from complex queries. The trade-off is eventual consistency: projections lag behind the source by seconds. We handle this by monitoring lag and setting user expectations.

### Trap questions (with answers)
1) **Q:** Why not just add indexes to the write database?
   - **A:** Indexes help, but complex queries (joins, aggregations, denormalization) still require computation at read time. Projections pre-compute the entire result. Also, heavy read queries can lock or slow the write DB; projections isolate reads.

2) **Q:** How do you prevent dual-write bugs?
   - **A:** Always use [Outbox Pattern](../architecture/outbox-pattern.md): write to source + outbox in a single transaction; a separate process reads outbox and emits events. This ensures atomicity between source write and event emission.

3) **Q:** What happens if a projection gets corrupted or out of sync?
   - **A:** Rebuild it from the source. If using event sourcing, replay all events to reconstruct the projection. If using CDC (Change Data Capture), reprocess the change log. This is why having a replayable source is critical.

4) **Q:** How do you handle schema changes in projections?
   - **A:** Blue-green deployment: create a new version of the projection with the new schema, replay events to populate it, then cutover once caught up. Keep the old projection until cutover completes.

5) **Q:** Can you have strong consistency between write and read models?
   - **A:** Not easily in distributed systems. Options: (1) read from source for critical operations (sacrifices speed), (2) synchronous projection update before ack (sacrifices availability), (3) accept eventual consistency (most common). Most systems choose (3) with low lag.

## Links
**Part of:**
- [System Design Archetypes](system-design-archetypes.md)

**Prerequisites:**
- [CQRS](../architecture/cqrs.md)
- [Event-Driven Architecture](../architecture/event-driven-basics.md)
- [Outbox Pattern](../architecture/outbox-pattern.md)

**Related archetypes:**
- [High-Throughput Event Ingestion](archetype-event-ingestion.md) — event source for projections
- [Search & Indexing Pipeline](archetype-search-indexing.md) — specialized type of projection
- [Time-Series Aggregation](archetype-time-series-aggregation.md) — windowed projections

**Related topics:**
- [Event Sourcing](../architecture/event-sourcing.md) — write model that enables easy rebuild
- [Eventual Consistency](../databases/eventual-consistency.md) (if exists)
