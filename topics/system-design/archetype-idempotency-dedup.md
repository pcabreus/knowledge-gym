---
id: archetype-idempotency-dedup
title: "Idempotency & Deduplication (Retry-Safe Writes)"
type: pattern
status: learning
importance: 95
difficulty: hard
tags: [system-design, idempotency, deduplication, exactly-once, retries]
canonical: true
aliases: [exactly-once-illusion, dedupe-window, request-dedup]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Idempotency & Deduplication (Retry-Safe Writes)

## TL;DR (BLUF)
- Make operations safe when clients retry or systems deliver messages multiple times.
- Key challenge: double-charging, double-shipping, duplicated records, race conditions around "first write wins".
- Solution patterns: idempotency keys + persistent record, unique constraints, transactional outbox.

## Definition
**What it is:** A system archetype that ensures operations produce the same result when executed multiple times, preventing duplicate side effects from retries or at-least-once delivery.

**Key terms:**
- **Idempotency**: Property where F(x) = F(F(x))—applying operation multiple times has same effect as once
- **Idempotency key**: Unique identifier (from client) to detect duplicate requests
- **Deduplication window**: Time period to track and reject duplicates
- **Exactly-once semantics**: Guarantee that each message/request is processed exactly once (practically: at-least-once + idempotent processing)

## Why it matters
- **Financial correctness**: Double-charging customers destroys trust and costs money
- **Data integrity**: Duplicate records corrupt analytics and business logic
- **User experience**: Accidental double-clicks shouldn't create duplicate orders
- **Interview frequency**: Critical for payments, orders, webhooks, async processing

**Typical real-world consequences of getting it wrong:**
- Payment provider times out after charging card; client retries → customer charged twice → chargebacks, support tickets, reputation damage
- Webhook delivered twice; order created twice → duplicate shipments, inventory errors
- Event reprocessed on retry → counters incremented twice, analytics wrong

## Scope & Non-goals
**In scope:**
- Idempotency key design and storage
- Deduplication strategies for writes
- Handling retries safely
- Side effect management (emails, payments)

**Out of scope / NOT solved by this:**
- Message delivery guarantees → see [Message Delivery Semantics](../architecture/message-delivery-semantics.md)
- Distributed transactions → see [Saga Pattern](../architecture/saga-pattern.md)
- Ordering guarantees → see [Ordering Guarantees](archetype-ordering-guarantees.md)

## Mental model / Intuition
Think of it like **form submission receipts**:
- Client submits form with tracking number (idempotency key)
- Server checks: "Did I already process this tracking number?"
- If yes → return the same result (no reprocessing)
- If no → process and store tracking number + result
- Even if client resubmits (refresh button), server uses stored result

## Decision rules (When to use / When not to use)
### Use it when
- **Side effects**: Payments, emails, inventory changes, webhooks
- **Client retries**: Network timeouts cause retries
- **At-least-once delivery**: Event systems may deliver duplicates
- **Critical correctness**: Financial, orders, legal compliance

### Avoid it when
- **Pure reads**: Idempotent by nature (GET requests)
- **Idempotent by design**: SET operations (not ADD/INCREMENT)
- **No retries**: Fully synchronous, reliable network (rare)

## How I would use it (practical)
### Context: Payment processing API
**Problem:** Client calls POST /payments. Network times out after payment provider charges card. Client retries → double charge.

**Steps:**
1. **Idempotency key**: Require clients to send `Idempotency-Key` header (UUID)
2. **Idempotency table**:
   ```sql
   CREATE TABLE idempotency_keys (
       key VARCHAR(255) PRIMARY KEY,
       response_status INT,
       response_body TEXT,
       created_at TIMESTAMP,
       expires_at TIMESTAMP
   );
   ```
3. **Processing flow**:
   - Check if key exists in table
   - If exists → return stored response (same status + body)
   - If not → process payment, store result in table with TTL (24h)
4. **Transaction**: Insert idempotency record + payment record in same DB transaction
5. **TTL**: Expire keys after 24h (clients shouldn't retry after that)

**What success looks like:**
- Zero double-charges (100% correctness)
- Retries return in < 50ms (cached result)
- Storage growth controlled by TTL

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Prevents duplicate side effects (correctness)
  - Safe retries enable reliability
  - Client-controlled retry safety
- **Cons / Risks:**
  - Storage overhead for idempotency records
  - Race conditions if not implemented atomically
  - Complexity in error scenarios (partial failures)

### Alternatives
- **No idempotency (YOLO):** Simple but dangerous
  - **When better:** Never for side effects; only for pure reads
- **Natural idempotency (unique constraints):** Simpler but not always possible
  - **When better:** Business key already unique (e.g., order_id); use DB constraint
- **Exactly-once delivery (Kafka transactions):** Offloads to infrastructure
  - **When better:** Pure event processing; no external side effects

**How to choose:**
- Side effects (payments, emails) → always implement idempotency keys
- Internal events → at-least-once + idempotent handlers
- Pure state updates → use natural unique constraints where possible

## Failure modes & Pitfalls
### Where it hurts (why it hurts)
1. **Double-charging, double-shipping, duplicated records**
   - **Symptom:** Customer charged twice, two orders created, duplicate shipments
   - **Why:** Retry without idempotency; side effects executed multiple times
   - **Solution:** Idempotency key + persistent record; return stored result on duplicate

2. **Race conditions around "first write wins"**
   - **Symptom:** Two concurrent requests with same key both process (double execution)
   - **Why:** Check-then-act not atomic; both see "key doesn't exist"
   - **Solution:** Use DB unique constraint or SELECT FOR UPDATE; atomic check-and-insert

3. **Storage overhead for idempotency records**
   - **Symptom:** Idempotency table grows unbounded; slow queries, disk full
   - **Why:** No TTL or cleanup; old keys accumulate
   - **Solution:** TTL (24h typical); background job to delete expired keys

4. **Side effects (payments, emails) need special care**
   - **Symptom:** Payment succeeds but DB write fails → no idempotency record → retry charges again
   - **Why:** Side effect not in same transaction as idempotency record
   - **Solution:** Two-phase: (1) reserve idempotency key, (2) execute side effect, (3) store result atomically; OR use [Outbox Pattern](../architecture/outbox-pattern.md)

## Observability (How to detect issues)
### Metrics
- `idempotency_key_duplicates_per_sec`: Retries detected
- `idempotency_key_race_conditions`: Concurrent requests with same key
- `idempotency_table_size`: Storage growth

### Logs
- Log duplicate: key, original_timestamp, response_reused

### Alerts
- `idempotency_key_race_conditions > 0` → atomicity bug
- `idempotency_table_size > 10M rows` → TTL not working

## Mini example (code)
```python
from db import transaction

def create_payment(idempotency_key, amount, customer):
    with transaction() as tx:
        # Check for existing key (atomic with SELECT FOR UPDATE)
        existing = tx.execute(
            "SELECT response_status, response_body FROM idempotency_keys "
            "WHERE key = ? FOR UPDATE",
            idempotency_key
        ).fetchone()
        
        if existing:
            # Return cached response
            return existing['response_body'], existing['response_status']
        
        # Process payment (may fail; transaction will rollback)
        payment_result = charge_card(amount, customer)
        
        # Store idempotency record + payment in same transaction
        tx.execute(
            "INSERT INTO idempotency_keys (key, response_status, response_body, expires_at) "
            "VALUES (?, ?, ?, NOW() + INTERVAL '24 hours')",
            idempotency_key, 200, payment_result
        )
        tx.execute(
            "INSERT INTO payments (id, amount, customer, status) VALUES (?, ?, ?, 'completed')",
            payment_result['id'], amount, customer
        )
        
        return payment_result, 200
```

## Common anti-patterns
### Anti-pattern: No idempotency for side effects
- **What people do:** Process payment/email on every request; rely on "clients won't retry"
- **Why it's bad:** Network timeouts force retries → duplicate side effects
- **Better approach:** Always use idempotency keys for non-idempotent operations

### Anti-pattern: Non-atomic check-and-insert
- **What people do:** Check if key exists; if not, process; then insert key (two separate queries)
- **Why it's bad:** Race condition—two concurrent requests both see "key doesn't exist"
- **Better approach:** Use DB unique constraint + INSERT ... ON CONFLICT or SELECT FOR UPDATE in transaction

### Anti-pattern: No TTL on idempotency keys
- **What people do:** Store keys forever "just in case"
- **Why it's bad:** Table grows unbounded; slows down; wastes storage
- **Better approach:** 24-48h TTL; clients shouldn't retry after that; if they do, treat as new request

### Anti-pattern: Storing result before executing side effect
- **What people do:** Insert idempotency key first, then call payment provider
- **Why it's bad:** If payment call fails, key is "used" but payment didn't happen; retry sees duplicate
- **Better approach:** Execute side effect first, then store key + result atomically; or use two-phase with status

## Interview readiness
### "Explain it like I'm teaching"
Idempotency makes operations safe to retry. When a client retries (due to timeout or error), we don't want to execute the operation twice—especially for things like payments or sending emails. We require clients to send an idempotency key (a unique ID they generate). On the first request, we process the operation and store the result with the key. On retries with the same key, we return the stored result without reprocessing. The hard part is handling race conditions: using database unique constraints or locks to ensure only one request with a given key is processed.

### Trap questions (with answers)
1) **Q:** Why not just rely on clients not to retry?
   - **A:** Network timeouts are unavoidable. Clients can't tell if a timeout means "request didn't reach server" or "server processed but response was lost." They must retry for reliability. Without idempotency, retries cause duplicates.

2) **Q:** How do you generate idempotency keys?
   - **A:** Client generates a UUID (or similar unique value) and includes it in requests. For retries of the same logical operation, client reuses the same key. Server doesn't generate keys—that would defeat the purpose (retries would look new).

3) **Q:** What's the difference between idempotency and deduplication?
   - **A:** Idempotency is a property: operation can be applied multiple times with same result. Deduplication is the mechanism: detecting and rejecting duplicate requests. We use deduplication (idempotency keys) to achieve idempotency.

4) **Q:** How do you handle side effects like payments that can't be in a DB transaction?
   - **A:** Two-phase: (1) Reserve the idempotency key (insert with status='processing'). (2) Execute side effect. (3) Update key with result (status='completed'). On retry, check status: if 'completed', return result; if 'processing', either wait or return error (concurrent request). Alternatively, use [Outbox Pattern](../architecture/outbox-pattern.md) to defer side effects.

5) **Q:** What if two requests with the same idempotency key arrive simultaneously?
   - **A:** Race condition. Use database atomicity: unique constraint (one will fail with conflict error) or SELECT FOR UPDATE lock (one waits for the other). Return the stored result to the second request.

## Links
**Part of:**
- [System Design Archetypes](system-design-archetypes.md)

**Prerequisites:**
- [Distributed Systems Basics](distributed-systems-basics.md)
- [Transactions](../databases/transactions.md)

**Related archetypes:**
- [High-Throughput Event Ingestion](archetype-event-ingestion.md) — at-least-once delivery requires idempotent consumers
- [Workflow Orchestration](archetype-workflow-orchestration.md) — idempotent steps
- [External Integrations](archetype-external-integrations.md) — webhooks and API retries

**Related topics:**
- [Message Delivery Semantics](../architecture/message-delivery-semantics.md)
- [Outbox Pattern](../architecture/outbox-pattern.md)
