---
id: archetype-event-ingestion
title: "High-Throughput Event Ingestion (Append-Only Log)"
type: pattern
status: learning
importance: 95
difficulty: hard
tags: [system-design, events, streaming, kafka, backpressure, throughput]
canonical: true
aliases: [event-ingestion-pipeline, streaming-ingestion, log-based-architecture, event-backbone]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# High-Throughput Event Ingestion (Append-Only Log)

## TL;DR (BLUF)
- Accept continuous streams of events (clicks, orders, sensors) and store/forward them reliably at scale.
- Key challenge: handling traffic spikes without dropping events while maintaining ordering and delivery guarantees.
- Solution patterns: partitioned logs (Kafka/Kinesis), backpressure, consumer groups, schema versioning.

## Definition
**What it is:** A system archetype that accepts a continuous, high-volume stream of events and reliably stores or forwards them to downstream consumers. Events are typically append-only and immutable.

**Key terms:**
- **Append-only log**: Immutable sequence of events
- **Partition**: Ordered subset of the log for parallelism
- **Partition key**: Attribute determining which partition an event goes to
- **Consumer lag**: How far behind consumers are from the latest event
- **Backpressure**: Mechanism to slow producers when consumers can't keep up

## Why it matters
- **Data loss prevention**: Spikes can cause dropped events → incorrect analytics, missing audit trails, lost business events
- **Downstream correctness**: Event order matters for state machines; reordering causes bugs
- **Schema evolution**: Producers and consumers evolve independently; incompatible changes break pipelines
- **Interview frequency**: Core to most streaming/event-driven system design questions

**Typical real-world consequences of getting it wrong:**
- E-commerce: Inventory updates lag during Black Friday; customers buy out-of-stock items
- Analytics: Missing conversion events lead to incorrect revenue reports
- Fraud detection: Delayed event processing allows fraudulent transactions to slip through

## Scope & Non-goals
**In scope:**
- Reliable event ingestion under variable load
- Partitioning strategy for ordering + parallelism
- Backpressure and consumer lag management
- Schema evolution without breaking consumers

**Out of scope / NOT solved by this:**
- Complex stream processing (joins, aggregations) → see [Time-Series Aggregation](archetype-time-series-aggregation.md)
- Exactly-once guarantees end-to-end → see [Idempotency & Deduplication](archetype-idempotency-dedup.md)
- Long-term storage/archival → see [Data Retention & Lifecycle](archetype-data-retention.md)

## Mental model / Intuition
Think of it like a **high-speed conveyor belt in a factory**:
- Events are items placed on the belt (producers)
- The belt has multiple parallel lanes (partitions)
- Items in the same lane stay in order
- Workers pick items from lanes (consumers)
- If workers slow down, the belt buffers items (backpressure)
- If items come too fast, you add more lanes or workers (scale partitions/consumers)

## Decision rules (When to use / When not to use)
### Use it when
- **High throughput**: 1000s-1000000s events/second
- **Multiple consumers**: Different services need the same events
- **Ordering matters**: Events for the same entity must be processed in order
- **Replay needed**: Consumers may need to reprocess historical events
- **Decoupling producers/consumers**: They evolve independently

### Avoid it when
- **Low volume**: < 100 events/second → simple queue or direct API calls may suffice
- **Strict transactional guarantees needed**: Append-only logs don't support rollback
- **Real-time strict latency**: < 10ms end-to-end → consider in-memory event buses
- **No need for replay**: If consumers only care about "latest state", simpler patterns work

## How I would use it (practical)
### Context: E-commerce order pipeline
**Problem:** Orders are placed, paid, shipped, delivered. Multiple services (inventory, notifications, analytics, fraud) need order events. During sales, order volume spikes 10×.

**Steps:**
1. **Choose partition key**: `order_id` → all events for one order stay ordered
2. **Set up Kafka topic**: `orders` with 10 partitions (scale for expected peak)
3. **Implement backpressure**: Producers fail fast if Kafka unavailable; retry with exponential backoff
4. **Schema Registry**: All events use Avro with schema versions; consumers declare compatible versions
5. **Consumer groups**: Each service has its own consumer group for independent lag tracking
6. **Monitor lag**: Alert if lag > 1 minute (indicator of consumer falling behind)

**What success looks like:**
- 99.99% event delivery (idempotent retries handle transient failures)
- p99 lag < 30s even during 10× spikes
- Zero schema-related consumer breaks during rollouts

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - High throughput and horizontal scalability
  - Decouples producers and consumers
  - Replay capability for recovery and new consumers
  - Durable (survives consumer/producer crashes)
- **Cons / Risks:**
  - Operational complexity (Kafka cluster management)
  - Eventual consistency (lag between write and consumer processing)
  - Ordering only per partition (not global)
  - Schema evolution requires discipline

### Alternatives
- **Direct API calls (request-response):** Simpler but couples services; no replay; hard to scale
  - **When better:** Low volume, tight coupling acceptable, no replay needed
- **Simple queue (SQS/RabbitMQ):** Easier operationally but limited replay, lower throughput
  - **When better:** < 10k events/sec, FIFO not critical, simpler ops preferred
- **Database as event log (outbox pattern):** Transactional safety but limited scale
  - **When better:** Need ACID guarantees, moderate volume

**How to choose:**
- Start with simple queue if < 10k events/sec and no replay needed
- Use append-only log (Kafka/Kinesis) when throughput, replay, or multiple consumers required
- Combine with [Outbox Pattern](../architecture/outbox-pattern.md) if producing from transactional DB

## Failure modes & Pitfalls
### Where it hurts (why it hurts)
1. **Traffic spikes (burst traffic)**
   - **Symptom:** Producer timeouts, dropped events, cascading failures
   - **Why:** Brokers/consumers can't absorb sudden 10× load
   - **Solution:** Backpressure (reject at edge), auto-scaling consumers, over-provision partitions

2. **Delivery guarantees confusion (at-least-once vs exactly-once)**
   - **Symptom:** Duplicate events processed, double-charging, incorrect counts
   - **Why:** Network retries cause duplicate delivery; "exactly-once" is complex/expensive
   - **Solution:** Default to at-least-once + idempotent consumers (see [Idempotency](archetype-idempotency-dedup.md))

3. **Schema evolution (producers/consumers drift)**
   - **Symptom:** Consumers crash on unknown fields, old consumers can't parse new events
   - **Why:** Producers deploy breaking changes without coordinating consumers
   - **Solution:** Schema Registry + backward/forward compatibility rules (Avro/Protobuf)

4. **Ordering issues (per-key vs global)**
   - **Symptom:** Events reordered (e.g., `OrderCanceled` processed before `OrderPlaced`)
   - **Why:** Different partitions or parallel consumers process out of order
   - **Solution:** Partition by entity ID for per-entity ordering; accept no global ordering

5. **Consumer lag explosion**
   - **Symptom:** Events processed hours/days late; downstream systems show stale data
   - **Why:** Slow consumer, poison message, under-provisioned consumer group
   - **Solution:** Lag monitoring + alerts, auto-scaling, DLQ for poison messages

## Observability (How to detect issues)
### Metrics
- **Producer:**
  - `produce_requests_failed` (failures to write)
  - `produce_latency_p99` (backpressure indicator)
  - `produce_bytes_per_sec` (throughput)
- **Broker (Kafka/Kinesis):**
  - `partition_count`, `partition_bytes`, `partition_lag`
  - `under_replicated_partitions` (reliability risk)
- **Consumer:**
  - `consumer_lag` (messages behind) — **critical metric**
  - `consumer_throughput` (messages/sec consumed)
  - `consumer_errors` (parse failures, processing errors)

### Logs
- Producer: Log event ID, partition, timestamp on send/ack
- Consumer: Log event ID, processing outcome (success/retry/DLQ)
- Schema errors: Log version mismatch details

### Traces
- Span: `produce_event` → `partition_assign` → `consume_event` → `process_event`
- Track latency from produce timestamp to consume timestamp

### Alerts
- `consumer_lag > 60s for 5 minutes` → consumers falling behind
- `produce_errors > 1% for 1 minute` → broker or backpressure issue
- `under_replicated_partitions > 0` → data loss risk

## Implementation notes
### Checklist
- [ ] Choose partition key (entity ID for ordering, or hash for distribution)
- [ ] Size partitions for peak load (2-3× normal to absorb spikes)
- [ ] Implement producer retries with backoff
- [ ] Set up Schema Registry with compatibility rules (backward or forward)
- [ ] Configure consumer groups (one per service)
- [ ] Set up consumer lag monitoring and alerting
- [ ] Define DLQ strategy for poison messages
- [ ] Plan partition expansion strategy (rebalancing is disruptive)

### Performance notes
- **Partition count**: More partitions = more parallelism but higher overhead; start with 10-50
- **Batch size**: Larger batches = higher throughput but higher latency; tune for p99 target
- **Replication factor**: 3 for production (survives 2 broker failures)

### Operational notes
- **Partition rebalancing**: Expensive; plan partition count for future growth
- **Retention**: Balance disk cost vs replay needs (7-30 days typical)
- **Monitoring**: Consumer lag is the most important metric

## Mini example (conceptual)
```python
# Producer side (Python/Kafka)
from kafka import KafkaProducer
import json

producer = KafkaProducer(
    bootstrap_servers=['kafka:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8'),
    acks='all',  # Wait for all replicas
    retries=3
)

def publish_order_event(order_id, event_type, payload):
    event = {
        'order_id': order_id,
        'event_type': event_type,
        'payload': payload,
        'timestamp': time.time()
    }
    # Partition by order_id for ordering
    producer.send('orders', key=order_id.encode('utf-8'), value=event)

# Consumer side
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'orders',
    bootstrap_servers=['kafka:9092'],
    group_id='inventory-service',
    enable_auto_commit=False,  # Manual commit after processing
    auto_offset_reset='earliest'
)

for message in consumer:
    event = json.loads(message.value)
    try:
        process_event(event)  # Idempotent processing
        consumer.commit()  # Only commit after success
    except Exception as e:
        log_error(e, event)
        send_to_dlq(event)  # Poison message handling
```

## Common anti-patterns
### Anti-pattern: No partition key (random distribution)
- **What people do:** Omit partition key, events distributed randomly
- **Why it's bad:** Breaks ordering for the same entity (e.g., order events out of order)
- **Better approach:** Always partition by entity ID for related events

### Anti-pattern: Synchronous fan-out from producer
- **What people do:** Producer calls multiple consumers directly before acking
- **Why it's bad:** Producer latency explodes; one slow consumer blocks all
- **Better approach:** Produce once to log; consumers pull independently

### Anti-pattern: Ignoring consumer lag
- **What people do:** No lag monitoring; discover lag days later when data is stale
- **Why it's bad:** Silent degradation; users see incorrect state
- **Better approach:** Monitor lag as a primary SLI; alert on > 60s lag

### Anti-pattern: Breaking schema changes without versioning
- **What people do:** Add required field to event schema; consumers crash
- **Why it's bad:** Zero-downtime rollouts become impossible
- **Better approach:** Use Schema Registry; only backward-compatible changes (optional fields)

## Interview readiness
### "Explain it like I'm teaching"
Event ingestion is about reliably accepting a continuous flood of events—like clicks, orders, or sensor readings—and making them available to multiple downstream systems. The core pattern is an append-only log (like Kafka) that acts as a buffer between producers and consumers. We partition the log so events for the same entity (like one order) stay in order, enabling parallelism without breaking correctness. When traffic spikes, backpressure prevents overload, and monitoring consumer lag tells us if downstream systems are keeping up.

### Trap questions (with answers)
1) **Q:** Why not just use a database for events?
   - **A:** Databases optimize for updates/queries, not append-only throughput. They also lack native backpressure and consumer-group semantics. However, for transactional writes, we combine DB + [Outbox Pattern](../architecture/outbox-pattern.md) to get ACID + event stream.

2) **Q:** How do you guarantee exactly-once delivery?
   - **A:** True exactly-once end-to-end is nearly impossible in distributed systems. Instead, we use at-least-once delivery (with retries) + idempotent consumers (dedupe on event ID). Some systems (Kafka with transactions) offer "effectively exactly-once" but it's complex and has trade-offs.

3) **Q:** What happens when you need to add more partitions?
   - **A:** Adding partitions causes rebalancing: consumers reconnect, and new partition assignment happens. This is disruptive (brief processing pause). Worse, existing data doesn't redistribute—only new events go to new partitions. Plan partition count for future growth upfront.

4) **Q:** How do you handle poison messages (events that crash consumers)?
   - **A:** Implement Dead Letter Queue (DLQ): after N failed processing attempts, move the event to a DLQ for manual inspection. Don't let one bad event block the entire partition. Also, ensure consumers have defensive parsing (catch parse errors separately).

5) **Q:** Why partition by entity ID instead of random distribution?
   - **A:** Random distribution maximizes parallelism but breaks ordering. If `OrderPaid` and `OrderShipped` for the same order land in different partitions, they can be processed out of order, corrupting state. Partitioning by `order_id` guarantees all events for one order are in-order, while still allowing parallelism across different orders.

## Links
**Part of:**
- [System Design Archetypes](system-design-archetypes.md)

**Prerequisites:**
- [Messaging Basics](../architecture/messaging-basics.md)
- [Message Delivery Semantics](../architecture/message-delivery-semantics.md)
- [Backpressure](backpressure.md)

**Related archetypes:**
- [Time-Series Aggregation](archetype-time-series-aggregation.md) — downstream processing of ingested events
- [Idempotency & Deduplication](archetype-idempotency-dedup.md) — handling duplicate event delivery
- [Ordering Guarantees](archetype-ordering-guarantees.md) — per-key sequencing patterns

**Patterns:**
- [Outbox Pattern](../architecture/outbox-pattern.md) — producing events from transactional DB
- [Event-Driven Architecture](../architecture/event-driven-basics.md)
- [Kafka](../architecture/kafka.md)

**Compares to:**
- [Request-Response](../architecture/request-response.md) — synchronous alternative
