---
id: backpressure
title: "Backpressure"
type: concept
status: learning
importance: 70
difficulty: medium
tags: [reliability, performance, concurrency, messaging]
canonical: true
aliases: []
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Backpressure

## TL;DR (BLUF)
- Backpressure limits incoming work when downstream capacity is saturated to keep the system stable.
- Use it with [Timeouts](../operations/timeouts.md), [Retries](../operations/retries-and-backoff.md), and [Circuit breaker](../operations/circuit-breaker.md).
- Trade-off: you may reject, delay, or degrade requests to protect throughput and latency.

## Definition
**What it is:** A flow-control mechanism that intentionally slows, buffers, or rejects work when consumers are overloaded, so the system stays within safe capacity bounds.  
**Key terms:** bounded concurrency, queue depth, [Load shedding](../operations/load-shedding.md), graceful degradation, consumer lag.

## Why it matters
- Prevents [Cascading failure](../operations/cascading-failure.md) when upstream producers outpace downstream capacity.
- Turns “slow meltdown” into controlled rejection or degraded service.

## Scope & Non-goals
**In scope:** limiting concurrency, queueing, shedding, and degrading to keep latency and error rates bounded.  
**Out of scope / NOT solved by this:** correctness under duplicates (see [Idempotency](../operations/idempotency.md)), or guaranteed delivery (see [Messaging basics](../architecture/messaging-basics.md)).

## Mental model / Intuition
- Think of a restaurant: if the kitchen is overloaded, you stop seating new tables, offer a reduced menu, or add a waitlist.
- Your goal is *stability first*, not maximum acceptance at any cost.

## Decision rules (When to use / When not to use)
### Use it when
- You have bursty traffic or uneven producer/consumer capacity.
- Downstream resources saturate (DB pools, CPU, external APIs).
- You need to protect latency SLOs and error budgets.

### Avoid it when
- The workload is trivial and capacity is far above peak.
- You cannot reject or delay work (rare; still prefer async buffering).

## How I would use it (practical)
- **Context:** An API triggers an async pipeline that calls 3 external services.
- **Steps:**
  1) Set bounded concurrency per worker group.
  2) Cap queue depth and reject/redirect excess load.
  3) Degrade responses (e.g., partial results) when saturated.
  4) Add explicit retry budgets to avoid load amplification.
- **What success looks like:** stable p95 latency and bounded queue lag under peak traffic.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** protects system health, improves tail latency under load, avoids cascading failure.
- **Cons / Risks:** rejected or delayed work, user-visible degradation, tuning complexity.
### Alternatives
- **Scale out:** add capacity if demand is consistently high.
- **Async decoupling:** shift work off the critical path with queues.
- **How to choose:** if overload is transient or spiky, backpressure is cheaper than scaling.

## Failure modes & Pitfalls
- Unbounded queues → latency spikes and memory pressure.
- Backpressure only at the edge → internal services still overload each other.
- Retrying without budgets → amplifies overload instead of relieving it.

## Observability (How to detect issues)
- **Metrics:** queue depth, consumer lag, worker utilization, CPU, p95/p99 latency, rejected requests.
- **Logs:** backpressure events (reason, threshold, current load).
- **Traces:** time spent waiting in queues vs processing.
- **Alerts:** queue depth > threshold, sustained saturation, rejection rate > X%.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define safe concurrency and queue limits per dependency
  - [ ] Decide rejection vs delay vs degrade
  - [ ] Enforce retry budgets to avoid amplification
  - [ ] Add per-tenant or per-route isolation
- **Go (practical)**
  - Use bounded goroutine pools, semaphores, or buffered channels.
  - Propagate cancellation with `context.Context` to stop wasted work.
- **Kafka consumers (practical)**
  - Tune `max.poll.records` and use `pause()`/`resume()` to match processing capacity.
  - Keep processing time below `max.poll.interval.ms` to avoid rebalances.
- **SQS consumers (practical)**
  - Limit in-flight messages and tune batch size per worker capacity.
  - Set visibility timeout to exceed worst-case processing time.
- **Security / Compliance notes**
  - Ensure degraded responses do not leak restricted data
- **Performance notes**
  - Prefer bounded queues over unbounded buffers
- **Operational notes**
  - Document overload playbook and thresholds

## Mini example (if applicable)
```go
// Bounded worker pool with backpressure via buffered channel.
const maxWorkers = 8
const maxQueue = 100

jobs := make(chan Job, maxQueue)

for i := 0; i < maxWorkers; i++ {
	go func() {
		for job := range jobs {
			process(job)
		}
	}()
}

func Enqueue(job Job) error {
	select {
	case jobs <- job:
		return nil
	default:
		return errors.New("queue full") // backpressure signal
	}
}
```

## Common anti-patterns
- **Anti-pattern:** “Just add retries when it’s slow.”
  - **Why it’s bad:** retries increase load and worsen overload.
  - **Better approach:** bound concurrency and reject or defer work.

## Interview readiness
### “Explain it like I’m teaching”
- Backpressure is a flow-control strategy: when consumers are saturated, you slow, buffer, or reject producers to keep the system within safe capacity and prevent cascading failure.

### Trap questions (with answers)
1) **Q:** Is a bigger queue always better for backpressure?
   - **A:** No. Unbounded queues hide overload and explode tail latency; bounded queues force explicit trade-offs.
2) **Q:** Can backpressure replace timeouts?
   - **A:** No. Backpressure protects capacity; timeouts bound how long work can consume resources.
3) **Q:** Does backpressure mean “drop all requests” under load?
   - **A:** Not necessarily. You can reject selectively, delay, or degrade depending on the SLA.

### Quick self-check (5 items)
- [ ] I can define backpressure precisely in 2–3 sentences.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Distributed systems basics](distributed-systems-basics.md)
- [Performance basics](performance-basics.md)
- [Observability basics](../operations/observability-basics.md)

### Related topics
- [Messaging basics](../architecture/messaging-basics.md)
- [Event-driven basics](../architecture/event-driven-basics.md)
- [Timeouts](../operations/timeouts.md)
- [Retries, exponential backoff, and jitter](../operations/retries-and-backoff.md)
- [Circuit breaker](../operations/circuit-breaker.md)
- [Load shedding](../operations/load-shedding.md)
- [Bulkheads](../operations/bulkheads.md)
- [Kafka](../architecture/kafka.md)
- (TODO) SQS/SNS
- (TODO) Rate limiting

### Compare with
- [Circuit breaker](../operations/circuit-breaker.md) — breaker blocks calls to a failing dependency; backpressure limits overall load.
- [Retries, exponential backoff, and jitter](../operations/retries-and-backoff.md) — retries add load; backpressure limits it.
- [Load shedding](../operations/load-shedding.md) — shedding drops/degrades work; backpressure slows or queues it.

## Notes / Inbox (optional)
- N/A
