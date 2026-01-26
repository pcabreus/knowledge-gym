---
id: data-reconciliation
title: "Data Reconciliation"
type: pattern
status: learning
importance: 75
difficulty: medium
tags: [operations, data-consistency, distributed-systems, reliability]
canonical: true
aliases: [reconciliation, data sync, consistency check]
created_at: 2026-01-26
updated_at: 2026-01-26
---

# Data Reconciliation

## TL;DR (BLUF)
- A process that detects and corrects data inconsistencies between systems by comparing states and resolving differences.
- Use it when eventual consistency is acceptable but divergence must be bounded and corrected.
- Trade-off: operational complexity and latency in detecting/fixing inconsistencies.

## Definition
**What it is:** A pattern where systems periodically or on-demand compare their data state with authoritative sources (or each other) to detect and correct inconsistencies that arise from failed writes, lost events, network partitions, or bugs.

**Key terms:** state comparison, drift detection, source of truth, compensating action, periodic reconciliation, on-demand reconciliation, data consistency, convergence.

## Why it matters
- Provides safety net for eventual consistency systems (event-driven, dual-write scenarios).
- Detects bugs in production (lost events, failed compensations, race conditions).
- Enables recovery from infrastructure failures (network partitions, service outages).
- Builds confidence in distributed systems despite lack of strong consistency.

## Scope & Non-goals
**In scope:** detecting inconsistencies, resolving conflicts, periodic checks, manual vs automated reconciliation.

**Out of scope / NOT solved by this:**
- Preventing inconsistencies (use [Outbox](../architecture/outbox-pattern.md), [Saga](../architecture/saga-pattern.md))
- Strong consistency (use ACID transactions within service)
- Real-time conflict resolution (reconciliation accepts lag)

## Mental model / Intuition
- Like balancing your bank account: periodically compare your records vs bank statement, identify differences, investigate and correct.
- In distributed systems: compare order-service DB vs payment-service DB; if order exists but payment missing → investigate and fix.

## Decision rules (When to use / When not to use)
### Use it when
- You have eventual consistency between systems (event-driven, dual-write, cache sync).
- Inconsistencies are rare but must be detected and fixed (not ignored).
- You can tolerate bounded staleness (minutes to hours).
- Systems can agree on "source of truth" or conflict resolution rules.
- Manual intervention for edge cases is acceptable.

### Avoid it when
- You need real-time consistency (use synchronous calls or distributed transactions).
- Reconciliation cost exceeds inconsistency cost (e.g., analytics where 99.9% accuracy is fine).
- You can't define source of truth or conflict resolution rules.
- Systems change too frequently (reconciliation always playing catch-up).

## How I would use it (practical)
- **Context:** E-commerce with order-service and payment-service. Events sync them, but occasionally events are lost.
- **Steps:**
  1) **Define source of truth:** payment-service is authoritative for payment status.
  2) **Schedule periodic reconciliation:** every 1 hour, compare last 24 hours of orders.
  3) **Detection logic:**
     ```sql
     SELECT o.order_id, o.payment_status, p.status
     FROM orders o
     LEFT JOIN payments p ON o.order_id = p.order_id
     WHERE o.payment_status != p.status
       OR (o.payment_status = 'PAID' AND p.id IS NULL)
     ```
  4) **Conflict resolution rules:**
     - If payment exists but order missing → create order (rare, investigate).
     - If order PAID but payment missing → mark order PENDING, alert ops (investigate).
     - If statuses differ → trust payment-service, update order-service.
  5) **Automated actions:**
     - Log discrepancies to dashboard.
     - Auto-fix simple cases (status mismatch).
     - Create tickets for manual review (missing records).
  6) **On-demand reconciliation:** Triggered after timeout on payment API call.
     ```go
     // After timeout on payment API
     if err == TimeoutError {
         // Don't retry; reconcile instead
         go reconcilePaymentStatus(orderID) // async status check
     }
     ```
  7) **Monitor:** reconciliation lag, discrepancy rate, auto-fix vs manual rate.

## Trade-offs / Costs (and their mitigation)
| Trade-off | Mitigation |
|-----------|-----------|
| Lag between inconsistency and detection | Run reconciliation more frequently; add on-demand triggers |
| Reconciliation job load on DBs | Run during off-peak; use read replicas; paginate queries |
| Conflict resolution ambiguity | Define clear rules; escalate ambiguous cases to manual review |
| False positives (transient inconsistencies) | Add grace period (e.g., ignore if < 5 min old) |
| Operational overhead (monitoring, alerts) | Automate simple fixes; build dashboards; set SLOs for reconciliation lag |

## Failure modes / Edge cases
1. **Reconciliation job fails:** Inconsistencies accumulate undetected.
   - *Mitigation:* Monitor job health; alert on failures; retry with backoff.
2. **Both systems wrong:** No clear source of truth.
   - *Mitigation:* Use event log as source of truth; replay events.
3. **Infinite reconciliation loop:** Auto-fix triggers opposite auto-fix.
   - *Mitigation:* Add idempotency; version data; prevent ping-pong with conflict flag.
4. **Reconciliation creates new inconsistency:** Fix introduces bug.
   - *Mitigation:* Test reconciliation logic thoroughly; dry-run mode; manual approval for large batches.
5. **High cardinality data:** Millions of records, slow reconciliation.
   - *Mitigation:* Incremental reconciliation (last N hours); use checksums/hashes for bulk comparison.

## Patterns
### 1. Periodic Reconciliation (Batch)
- Scheduled job (cron, Airflow) compares full dataset or time window.
- **Pros:** Simple, comprehensive.
- **Cons:** High latency (hours), resource-intensive.
- **Use:** Nightly reconciliation of orders, inventory, billing.

### 2. On-Demand Reconciliation (Reactive)
- Triggered by specific events (timeout, user complaint, suspicious pattern).
- **Pros:** Fast for critical cases.
- **Cons:** Doesn't catch unknown issues.
- **Use:** After payment timeout, check payment status before retrying.

### 3. Continuous Reconciliation (Stream)
- Real-time comparison using event streams (Kafka, Change Data Capture).
- **Pros:** Low latency (seconds).
- **Cons:** Complex, higher resource usage.
- **Use:** High-value transactions, real-time fraud detection.

### 4. Checksum-based Reconciliation
- Compare aggregate checksums (hash of dataset) instead of row-by-row.
- **Pros:** Fast for large datasets.
- **Cons:** Doesn't identify specific inconsistencies.
- **Use:** Detect drift; drill down only if checksums differ.

## Conflict Resolution Strategies
1. **Source of truth wins:** One system is authoritative (simplest).
2. **Last write wins (LWW):** Use timestamp; newest update wins (Cassandra, DynamoDB).
3. **Manual resolution:** Escalate to ops/support for human decision.
4. **Merge / union:** Combine both versions if not conflicting (CRDTs).
5. **Compensating action:** Trigger saga compensation (e.g., refund + cancel order).

## Combinations
Data Reconciliation is **almost always used with:**
- **[Idempotency](idempotency.md):** Reconciliation actions must be safe to retry.
- **[Outbox pattern](../architecture/outbox-pattern.md):** Reduces need for reconciliation (reliable events).
- **[Saga](../architecture/saga-pattern.md):** Reconcile failed compensations.
- **[Observability](observability-basics.md):** Monitor drift, reconciliation lag, fix rate.
- **[Change Data Capture](../databases/change-data-capture.md):** Stream changes for continuous reconciliation.

**Typical combination:**
- **Event-driven system:** Outbox (reduce drift) + Idempotency (safe fixes) + Reconciliation (safety net) + Observability

## Prerequisites
- Understanding of [Eventual consistency](../databases/consistency-models.md).
- Familiarity with [Event-driven architecture](../architecture/event-driven-basics.md).
- Knowledge of [Dual-write pattern](../architecture/dual-write-pattern.md) and its problems.

## Related topics
- [Outbox pattern](../architecture/outbox-pattern.md): Prevents inconsistencies proactively.
- [Saga pattern](../architecture/saga-pattern.md): Reconcile failed compensations.
- [Dual-write pattern](../architecture/dual-write-pattern.md): Common source of drift.
- [Idempotency](idempotency.md): Required for safe reconciliation.
- [Change Data Capture](../databases/change-data-capture.md): Stream-based reconciliation.
- [Observability](observability-basics.md): Detect drift early.

## Real-world examples
1. **Stripe:** Reconciles payment events with bank settlement files daily.
2. **Amazon:** Reconciles order status across warehouses, inventory, and shipping.
3. **Banking:** ATM transactions reconciled nightly with core banking system.
4. **Netflix:** Reconciles user viewing history across regional data centers.

## Checklist (self-test)
- [ ] I can identify when reconciliation is needed vs when prevention is better.
- [ ] I know how to define source of truth and conflict resolution rules.
- [ ] I can design periodic vs on-demand vs continuous reconciliation.
- [ ] I understand the trade-off between reconciliation frequency and cost.
- [ ] I can monitor reconciliation health (lag, discrepancy rate, fix rate).

## Reminders / Key takeaways
- Reconciliation is a **safety net**, not a primary strategy (prevent inconsistencies first).
- Always define **source of truth** and **conflict resolution rules** upfront.
- Use **[Idempotency](idempotency.md)** for reconciliation actions (safe to retry).
- Monitor **drift rate** and **reconciliation lag** (SLOs).
- Combine with **[Outbox](../architecture/outbox-pattern.md)** to minimize need for reconciliation.

## Trap questions (with answers)
### Q1: Should I use reconciliation instead of fixing the dual-write problem?
**A:** **No**. Reconciliation is a **safety net**, not a fix. First, eliminate dual-write using [Outbox pattern](../architecture/outbox-pattern.md) or [Change Data Capture](../databases/change-data-capture.md). Then add reconciliation to catch edge cases (bugs, infrastructure failures). Relying solely on reconciliation means accepting frequent inconsistencies.

### Q2: How often should I run reconciliation?
**A:** **Depends on tolerance for drift**. Guidelines:
- **Critical data (payments, orders):** Every 5-15 minutes OR on-demand after timeouts.
- **Important data (inventory, user profiles):** Hourly.
- **Analytics, caches:** Daily or weekly.
Balance: frequency vs DB load. Monitor **drift rate**—if high, fix root cause, don't just reconcile faster.

### Q3: What if reconciliation finds conflicting data in both systems?
**A:** **Define conflict resolution rules**:
1. **Source of truth:** One system is authoritative (simplest).
2. **Last write wins:** Use timestamps (beware clock skew).
3. **Manual review:** Escalate to ops (for complex cases).
4. **Event log:** Replay events to reconstruct correct state.
If no rule applies, log for manual investigation. Never silently pick one—this hides bugs.

### Q4: Can reconciliation replace idempotency?
**A:** **No, they're complementary**. **[Idempotency](idempotency.md)** ensures **duplicate requests don't cause side effects**. **Reconciliation** **detects and fixes inconsistencies** that already occurred. You need both: idempotency prevents duplicates during reconciliation; reconciliation catches cases where idempotency failed (e.g., TTL expired).

### Q5: After a timeout on a payment API call, should I retry or reconcile?
**A:** **Reconcile first, then decide**. Timeout means you don't know if payment succeeded. Options:
1. **Check status** (GET /payments/{id}) using [Idempotency](idempotency.md) key.
2. If "Processing" → wait/poll (don't retry POST).
3. If "Success" → mark order paid (no retry).
4. If "Failed" → safe to retry POST.
Never blindly retry POST after timeout—risks double charge. See [Timeouts](timeouts.md) and [Third-party integration](../operations/third-party-integration-resilience.md).
