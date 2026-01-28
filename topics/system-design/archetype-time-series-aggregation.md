---
id: archetype-time-series-aggregation
title: "Time-Series + Windowed Aggregation"
type: pattern
status: learning
importance: 90
difficulty: hard
tags: [system-design, time-series, windowing, aggregation, streaming, metrics]
canonical: true
aliases: [time-series-aggregation, windowing, ohlc-candles, bucketization, rollups, downsampling]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Time-Series + Windowed Aggregation

## TL;DR (BLUF)
- Transform raw time-stamped events into aggregates per time window (1s/1m/5m/1h), often hierarchical.
- Key challenge: late/out-of-order events, high cardinality, retention costs, real-time vs correctness trade-off.
- Solution patterns: stream processing with watermarks, materialized views, hierarchical rollups, time-series DBs.

## Definition
**What it is:** A system archetype that continuously aggregates time-stamped events into fixed or sliding time windows, producing metrics like counts, sums, averages, or complex aggregates (OHLC candles in finance).

**Key terms:**
- **Tumbling window**: Fixed non-overlapping time window (e.g., every 1 minute)
- **Sliding window**: Overlapping windows (e.g., 5-minute window sliding every 1 minute)
- **Watermark**: Timestamp indicating "events before this are complete"
- **Late event**: Event arriving after its window has closed
- **Rollup**: Pre-aggregated data at coarser granularity (1m → 5m → 1h)
- **Cardinality**: Number of unique dimension combinations (symbol × exchange × user)

## Why it matters
- **Real-time dashboards**: Users expect metrics within seconds (monitoring, trading, IoT)
- **Cost control**: Raw data is huge; queries on raw data are slow and expensive
- **Correctness vs latency**: Late events break window correctness if not handled
- **Interview frequency**: Common in streaming, monitoring, analytics system design

**Typical real-world consequences of getting it wrong:**
- Trading app: Late ticks cause candle revisions → user trust broken, compliance issues
- Monitoring: Alert delays or wrong thresholds due to aggregation lag
- IoT: High cardinality (millions of devices) explodes memory → OOM crashes

## Scope & Non-goals
**In scope:**
- Window types (tumbling, sliding, session)
- Handling late/out-of-order events
- Hierarchical rollups for retention and query speed
- High cardinality management

**Out of scope / NOT solved by this:**
- Event ingestion → see [High-Throughput Event Ingestion](archetype-event-ingestion.md)
- Long-term archival → see [Data Retention & Lifecycle](archetype-data-retention.md)
- Complex event correlation (joins) → stream processing topic

## Mental model / Intuition
Think of it like **weather reporting stations**:
- Raw events: temperature sensor readings every second (massive data)
- Tumbling windows: "Average temperature per hour" (bucket)
- Sliding windows: "Last 10-minute average, updated every minute"
- Late events: Delayed sensor data arrives; do you revise the hour's average or ignore it?
- Rollups: Keep hourly averages forever, but only raw data for 24 hours

## Decision rules (When to use / When not to use)
### Use it when
- **Time-based queries dominate**: "Show me metrics for last 7 days"
- **High event volume**: Raw queries are too slow/expensive
- **Multiple time granularities**: Users need 1m, 1h, 1d views
- **Real-time or near-real-time**: Dashboards, alerts, live metrics

### Avoid it when
- **Low volume**: < 100 events/sec → query raw data directly
- **No time-based queries**: Data is not time-series in nature
- **Perfect accuracy required**: Windowing involves trade-offs (late events)

## How I would use it (practical)
### Context: Trading platform tick data → candles
**Problem:** Receive tick data `(symbol, price, volume, timestamp)` at 100k ticks/sec. Users request 1-minute OHLC (Open/High/Low/Close) candles for charting.

**Steps:**
1. **Choose window type**: Tumbling 1-minute windows (no overlap)
2. **Set watermark policy**: Allow 10s lateness (most late ticks arrive within 10s)
3. **Stream processor (Flink/Kafka Streams)**:
   - Key by `symbol`
   - Window: tumbling 1-minute
   - Aggregate: compute OHLC + volume
   - Watermark: event time + 10s lag tolerance
4. **Handle late events**: Option A: discard (fast, simple); Option B: update candle (complex, confusing for users)
5. **Hierarchical rollups**:
   - Keep raw ticks: 24h
   - 1m candles: 30 days
   - 5m candles: 1 year
   - 1h candles: forever
6. **Store in time-series DB** (TimescaleDB / InfluxDB) for fast range queries

**What success looks like:**
- Candles available within 1 second of window close
- < 0.1% late events discarded (most arrive within watermark)
- Query p99 < 200ms for 1-day range

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Orders of magnitude faster queries vs raw data
  - Controllable retention costs (raw → rollups)
  - Real-time metrics with tunable latency
- **Cons / Risks:**
  - Late events require trade-off: accuracy vs latency
  - High cardinality (many dimensions) explodes state/memory
  - Watermark tuning: too short = miss events; too long = high latency

### Alternatives
- **Query raw data on demand:** Simple but slow and expensive at scale
  - **When better:** < 100 events/sec, infrequent queries
- **Batch aggregation (hourly/daily jobs):** Lower operational cost but not real-time
  - **When better:** Dashboards tolerate hours of staleness
- **Write-time aggregation (increment counters):** Fast reads but inflexible (can't change dimensions)
  - **When better:** Fixed set of dimensions, simple aggregates (counts/sums)

**How to choose:**
- Real-time dashboards + high volume → stream processing with windows
- Historical analysis only → batch aggregation
- Fixed metrics (e.g., "total orders") → write-time aggregation

## Failure modes & Pitfalls
### Where it hurts (why it hurts)
1. **Late/out-of-order events break window correctness**
   - **Symptom:** Candles or metrics change retroactively; users see inconsistent data
   - **Why:** Network jitter, clock skew, replay cause events to arrive late
   - **Solution:** Watermarks with allowed lateness; choose "discard" or "update" policy explicitly

2. **High cardinality (many symbols/users/devices) explodes memory/state**
   - **Symptom:** OOM crashes, spilling to disk, degraded performance
   - **Why:** Maintaining state per `(symbol, user, device)` combination scales poorly
   - **Solution:** Sample high-cardinality dimensions, approximate sketches (HyperLogLog), cap dimensions

3. **Retention: raw data is huge; queries must stay fast**
   - **Symptom:** Disk full, slow queries, expensive storage
   - **Why:** Keeping all raw data is impractical; but users want historical analysis
   - **Solution:** Hierarchical rollups (1m → 5m → 1h) with tiered retention policies

4. **Real-time vs correctness tension**
   - **Symptom:** Metrics update instantly but miss late data; or metrics are accurate but delayed
   - **Why:** Watermark tuning trade-off
   - **Solution:** Emit "preliminary" results early + "final" results after watermark; let users choose

## Observability (How to detect issues)
### Metrics
- `window_close_latency_p99`: Time from window end to result emission
- `late_events_dropped`: Count of events arriving after watermark
- `window_state_size_bytes`: Memory usage for active windows
- `cardinality (unique_keys)`: Track dimension explosion

### Logs
- Log late event: timestamp, window, watermark, action (discard/update)
- Log window close: window_id, count, aggregate values

### Traces
- Span: `event_arrive` → `window_assign` → `aggregate` → `window_emit`

### Alerts
- `late_events_dropped > 1% for 5 minutes` → watermark too aggressive
- `window_state_size > 80% memory` → cardinality explosion or leak
- `window_close_latency > 10s` → processing backlog

## Implementation notes
### Checklist
- [ ] Choose window type (tumbling / sliding / session)
- [ ] Define watermark policy (allowed lateness)
- [ ] Decide late event handling (discard / update / both)
- [ ] Plan rollup hierarchy (raw → 1m → 5m → 1h)
- [ ] Set retention per granularity
- [ ] Monitor cardinality and cap if needed
- [ ] Choose time-series DB or materialized view storage

### Performance notes
- **Watermark tuning**: 5-10s typical for most event sources; tune based on p99 event delay
- **State size**: Proportional to cardinality × windows open; use session windows cautiously
- **Rollup writes**: Pre-aggregate during window close to avoid recalculating on read

### Operational notes
- **Reprocessing**: If logic changes, replay raw events to rebuild rollups
- **Compaction**: Merge small rollup chunks to reduce query overhead (time-series DB feature)

## Mini example (conceptual)
```python
# Flink-style pseudocode
events = stream.from_kafka('sensor_readings')

# Tumbling 1-minute windows, 10s allowed lateness
windowed = events \
    .key_by(lambda e: e.device_id) \
    .window(TumblingWindow.of_minutes(1)) \
    .allowed_lateness(seconds(10)) \
    .aggregate(
        lambda acc, e: {
            'count': acc['count'] + 1,
            'sum': acc['sum'] + e.value,
            'max': max(acc['max'], e.value)
        },
        initial={'count': 0, 'sum': 0, 'max': -inf}
    )

# Emit to time-series DB
windowed.sink_to_timescaledb('metrics_1m')

# Hierarchical rollup: 1m → 5m
rollup_5m = windowed \
    .window(TumblingWindow.of_minutes(5)) \
    .aggregate(lambda acc, e: {...})
```

## Common anti-patterns
### Anti-pattern: No watermark / immediate close
- **What people do:** Close window immediately when clock hits window end
- **Why it's bad:** Late events (network jitter) get dropped or cause inconsistency
- **Better approach:** Allow 5-10s lateness via watermark

### Anti-pattern: Unbounded allowed lateness
- **What people do:** Accept events no matter how late to avoid data loss
- **Why it's bad:** Windows never close; memory leaks; metrics perpetually "in progress"
- **Better approach:** Cap lateness (e.g., 1 minute); DLQ for extremely late events

### Anti-pattern: No rollups / query raw data
- **What people do:** Keep all raw events forever; query on demand
- **Why it's bad:** Storage explodes; queries become unusably slow
- **Better approach:** Raw data short retention (7d); pre-aggregate rollups for longer retention

### Anti-pattern: High-cardinality dimensions without sampling
- **What people do:** Window by `(user_id, device_id, endpoint, status)`
- **Why it's bad:** Millions of unique keys → OOM
- **Better approach:** Sample rare dimensions, use approximate sketches (HyperLogLog for cardinality)

## Interview readiness
### "Explain it like I'm teaching"
Time-series aggregation is about turning a flood of raw time-stamped events into useful metrics over time windows. For example, millions of stock ticks per second become 1-minute OHLC candles. The hard parts are: (1) handling late events without breaking correctness or adding too much latency, (2) managing high cardinality (e.g., millions of devices) without running out of memory, and (3) balancing retention costs by pre-aggregating into hierarchical rollups—keep raw data for days, 1-minute rollups for months, hourly rollups forever.

### Trap questions (with answers)
1) **Q:** Why not just compute aggregates on-demand from raw data?
   - **A:** At scale, querying raw data for "last 7 days" on millions of events/sec is too slow and expensive. Pre-aggregating trades freshness/flexibility for performance and cost.

2) **Q:** What's the difference between tumbling and sliding windows?
   - **A:** Tumbling windows are non-overlapping fixed intervals (e.g., 00:00-01:00, 01:00-02:00). Sliding windows overlap (e.g., "last 5 minutes" computed every 1 minute: 00:00-05:00, 01:00-06:00). Sliding windows are more expensive (more windows active) but provide smoother metrics.

3) **Q:** How do you decide the watermark lateness value?
   - **A:** Measure p99 event delay in your system (time between event timestamp and arrival). Set watermark to slightly above that (e.g., if p99 delay is 5s, use 10s watermark). Monitor late events dropped; if > 1%, increase watermark.

4) **Q:** What do you do when a late event arrives after the window closed and watermark passed?
   - **A:** Two options: (1) Discard it (fast, simple, data loss). (2) Update the window retroactively (accurate but confusing for users; requires versioning or "final" flag). Most systems choose (1) with low watermark tolerance to minimize loss.

5) **Q:** How do hierarchical rollups work?
   - **A:** Compute coarser-grained aggregates from finer-grained ones: 1-minute → 5-minute → 1-hour. Retain raw events for 7 days, 1m for 30 days, 5m for 1 year, 1h forever. Queries hit the coarsest rollup that satisfies the range. This saves storage and speeds up queries.

## Links
**Part of:**
- [System Design Archetypes](system-design-archetypes.md)

**Prerequisites:**
- [High-Throughput Event Ingestion](archetype-event-ingestion.md)
- [Distributed Systems Basics](distributed-systems-basics.md)

**Related archetypes:**
- [Materialized Views / CQRS](archetype-materialized-views.md) — precomputed read models
- [Data Retention & Lifecycle](archetype-data-retention.md) — tiered storage
- [Search & Indexing Pipeline](archetype-search-indexing.md) — another precomputation pattern

**Related topics:**
- [Time-series databases](../databases/time-series-databases.md) (if exists)
- [Streaming processing](../architecture/streaming-processing.md) (if exists)
