---
id: backpressure-mcq-1
topic: ../topics/system-design/backpressure.md
kind: mcq
difficulty: medium
timebox_minutes: 20
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Backpressure — MCQ 1

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
Q1) What is backpressure in a distributed system?
A) A flow-control strategy that limits, delays, or rejects work when consumers are saturated.
B) A database replication mechanism that guarantees exactly-once delivery.
C) A UI animation technique that smooths scrolling under load.
D) A caching policy that increases TTL under high traffic.

Q2) What is the most likely impact of removing backpressure from an async pipeline that already uses retries?
A) Faster average latency and fewer resource spikes.
B) Queue growth, higher tail latency, and potential memory pressure.
C) Lower error rates because retries will succeed more often.
D) The system becomes “eventually consistent” by default.

Q3) You have a synchronous API that calls a database with a small connection pool. Which design best adds backpressure?
A) Spawn a goroutine per request and rely on GC to keep up.
B) Add a bounded worker pool and return 429/503 when the queue is full.
C) Increase client timeouts so requests wait longer.
D) Disable database pooling to reduce contention.

Q4) What is the primary trade-off of using a very large queue as your only backpressure mechanism?
A) It reduces memory usage and lowers tail latency.
B) It hides overload, increases tail latency, and risks resource exhaustion.
C) It guarantees exactly-once processing.
D) It eliminates the need for retries and timeouts.

Q5) In Go, which implementation most directly enforces backpressure on a CPU-bound worker group?
A) Launch one goroutine per job with no limit.
B) Use a bounded worker pool and a buffered channel with a drop/reject path.
C) Increase GOMAXPROCS to the number of incoming requests.
D) Replace goroutines with reflection to reduce overhead.

Q6) A Kafka consumer is falling behind and triggering rebalances due to long processing. Which backpressure-oriented change is best?
A) Increase `max.poll.records` and let the consumer handle more at once.
B) Use `pause()`/`resume()` and reduce `max.poll.records` to match processing capacity.
C) Disable commits so processing is faster.
D) Delete the topic to remove lag.

Q7) For SQS consumers, which change most directly applies backpressure?
A) Increase the receive batch size and spawn unlimited workers.
B) Limit in-flight messages and tune visibility timeout to match processing time.
C) Disable long polling to get faster responses.
D) Use FIFO queues because they are always faster.

Q8) When should you avoid backpressure and instead focus on scaling capacity?
A) When overload is transient and spiky.
B) When demand is consistently above capacity and work cannot be dropped or delayed.
C) When the system is idle and has spare capacity.
D) When the system uses HTTP/2.

Q9) Which statement best differentiates backpressure from a circuit breaker?
A) Backpressure limits load even when dependencies are healthy; a breaker trips on unhealthy dependencies.
B) Backpressure guarantees exactly-once delivery; a breaker guarantees at-least-once delivery.
C) Backpressure is only for databases; breakers are only for HTTP.
D) Backpressure is a logging feature; breakers are a monitoring feature.

Q10) Your service supports a “lite” response that omits optional data. During overload, which policy best applies graceful degradation as part of backpressure?
A) Return a 200 with a reduced payload and clear indicators of omitted data.
B) Retry the same request indefinitely until full data is available.
C) Accept all requests and let queues grow unbounded.
D) Respond with random data to keep latency low.

## Answer format (what “good” looks like)
- Correct letter per question
- Short justification focused on capacity, latency, and failure modes

## Answer key / Rubric
### Correct answer (for MCQ)
Q1)
- Correct: A
- Why correct: Backpressure is explicit flow control that limits, delays, or rejects work when consumers are saturated to protect system stability.
- Why A is wrong: N/A
- Why B is wrong: Replication and delivery guarantees are different concerns.
- Why C is wrong: UI animation is unrelated to system flow control.
- Why D is wrong: TTL policies are caching, not overload control.

Q2)
- Correct: B
- Why correct: Without backpressure, retries can amplify load, increasing queue depth, tail latency, and memory pressure.
- Why A is wrong: Removing backpressure rarely improves stability under load.
- Why C is wrong: Retries can worsen overload; they don’t reduce errors by themselves.
- Why D is wrong: Consistency models are unrelated to queue overload.

Q3)
- Correct: B
- Why correct: Bounded concurrency with a capped queue prevents DB pool saturation and provides explicit overload signaling.
- Why A is wrong: Unbounded goroutines amplify contention and collapse the pool.
- Why C is wrong: Longer timeouts increase contention and latency.
- Why D is wrong: Disabling pooling increases connection churn and failure rates.

Q4)
- Correct: B
- Why correct: Large queues hide overload, push latency to the tail, and can exhaust memory, causing worse failures.
- Why A is wrong: Large queues increase memory usage and tail latency.
- Why C is wrong: Queues do not guarantee exactly-once processing.
- Why D is wrong: Queues don’t eliminate retries/timeouts; they are complementary.

Q5)
- Correct: B
- Why correct: A bounded worker pool and buffered channel enforce a hard cap on concurrency and provide a rejection path.
- Why A is wrong: Unbounded goroutines remove flow control.
- Why C is wrong: GOMAXPROCS doesn’t bound request concurrency.
- Why D is wrong: Reflection is unrelated to flow control and performance.

Q6)
- Correct: B
- Why correct: Pausing consumption and lowering batch sizes aligns ingestion with processing capacity, reducing lag and rebalances.
- Why A is wrong: Increasing poll size can worsen lag if processing is slow.
- Why C is wrong: Disabling commits doesn’t fix capacity and risks reprocessing.
- Why D is wrong: Deleting the topic is absurd and destroys data.

Q7)
- Correct: B
- Why correct: Limiting in-flight messages and matching visibility timeout to processing time provides controlled concurrency and avoids redelivery storms.
- Why A is wrong: Unlimited workers remove backpressure and create overload.
- Why C is wrong: Disabling long polling increases empty receives and cost.
- Why D is wrong: FIFO is about ordering and dedup, not throughput or backpressure.

Q8)
- Correct: B
- Why correct: If demand consistently exceeds capacity and you cannot drop or delay, scaling is the durable fix.
- Why A is wrong: Transient spikes are exactly when backpressure helps.
- Why C is wrong: Idle systems don’t need special policies to accept load.
- Why D is wrong: HTTP/2 doesn’t determine backpressure needs.

Q9)
- Correct: A
- Why correct: Backpressure limits load based on capacity, while a breaker trips on observed dependency failures.
- Why A is wrong: N/A
- Why B is wrong: Delivery guarantees are unrelated to breakers or backpressure.
- Why C is wrong: Both can apply across many dependency types.
- Why D is wrong: Neither is a logging or monitoring feature.

Q10)
- Correct: A
- Why correct: Graceful degradation reduces load while preserving user value and explicit signaling.
- Why A is wrong: N/A
- Why B is wrong: Infinite retries amplify load and hurt latency.
- Why C is wrong: Unbounded queues worsen overload.
- Why D is wrong: Returning random data is nonsensical and unsafe.

## Common mistakes
- Treating backpressure as just “add a queue.”
- Ignoring retry budgets.
- Forgetting per-tenant or per-route isolation.
