---
id: bulkheads-mcq-1
topic: ../topics/operations/bulkheads.md
kind: mcq
difficulty: medium
timebox_minutes: 20
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Bulkheads — MCQ 1

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
Q1) What problem do bulkheads primarily solve?
A) Preventing one component or tenant from exhausting shared resources.
B) Providing exactly-once delivery guarantees.
C) Compressing network payloads to reduce latency.
D) Ensuring schema migrations are backward compatible.

Q2) Which example best illustrates a bulkhead?
A) Separate thread pools for a flaky dependency and a critical dependency.
B) A global retry loop for all requests.
C) A single shared queue for all endpoints.
D) A cron job that runs hourly.

Q3) How do bulkheads differ from circuit breakers?
A) Bulkheads isolate capacity; breakers stop calls to unhealthy dependencies.
B) Bulkheads control cache eviction; breakers control cache invalidation.
C) Bulkheads are about HTTP headers; breakers are about DNS.
D) Bulkheads guarantee delivery; breakers guarantee ordering.

Q4) What is a key trade-off of bulkheads?
A) Capacity fragmentation and potential under-utilization.
B) Guaranteed higher throughput in all cases.
C) Elimination of the need for observability.
D) Automatic resolution of dependency failures.

Q5) When should you avoid fine-grained bulkheads?
A) When partitioning would create many tiny pools with high idle waste.
B) When you have multi-tenant workloads.
C) When dependencies have different risk profiles.
D) When you need isolation against noisy neighbors.

Q6) Which metric best indicates a bulkhead sizing issue?
A) One pool is saturated while others are idle.
B) All pools are perfectly balanced at 50% utilization.
C) Cache hit ratio spikes.
D) TLS handshake time decreases.

Q7) How do bulkheads and backpressure relate?
A) Bulkheads isolate capacity by component; backpressure limits overall load.
B) Bulkheads replace the need for backpressure entirely.
C) Backpressure guarantees delivery; bulkheads drop requests.
D) They are the same mechanism with different names.

Q8) Which change is most likely to improve bulkhead effectiveness in a multi-tenant system?
A) Add per-tenant concurrency caps within each pool.
B) Remove all tenant limits to maximize throughput.
C) Disable authentication to reduce CPU.
D) Merge all pools into one to simplify configuration.

Q9) What is a common failure mode of bulkheads?
A) Over-partitioning that wastes capacity and adds tuning overhead.
B) Guaranteeing too much availability.
C) Turning retries into circuit breakers.
D) Eliminating the need for rate limits.

Q10) Which combination is most defensible for a critical dependency?
A) Bulkheads + timeouts + circuit breaker.
B) Only retries with no timeouts.
C) Only a large shared pool.
D) Disabling monitoring to reduce noise.

## Answer format (what “good” looks like)
- Correct letter per question
- Short justification focused on isolation, capacity, and failure modes

## Answer key / Rubric
### Correct answer (for MCQ)
Q1)
- Correct: A
- Why correct: Bulkheads isolate resources so one component or tenant can’t exhaust shared capacity.
- Why A is wrong: N/A
- Why B is wrong: Delivery guarantees are unrelated to bulkheads.
- Why C is wrong: Compression doesn’t address resource isolation.
- Why D is wrong: Schema compatibility is unrelated.

Q2)
- Correct: A
- Why correct: Separate pools isolate risk and prevent a flaky dependency from starving critical work.
- Why A is wrong: N/A
- Why B is wrong: Global retries amplify overload and do not isolate capacity.
- Why C is wrong: A single queue couples all workloads.
- Why D is wrong: Cron jobs don’t provide isolation.

Q3)
- Correct: A
- Why correct: Bulkheads contain failures by isolating capacity; breakers stop calls to unhealthy dependencies.
- Why A is wrong: N/A
- Why B is wrong: Cache control is unrelated.
- Why C is wrong: HTTP/DNS are unrelated.
- Why D is wrong: Neither guarantees delivery or ordering.

Q4)
- Correct: A
- Why correct: Partitioning resources can leave capacity idle and requires tuning.
- Why A is wrong: N/A
- Why B is wrong: Bulkheads don’t guarantee higher throughput.
- Why C is wrong: Observability is still required.
- Why D is wrong: Bulkheads don’t fix dependency health.

Q5)
- Correct: A
- Why correct: Too many tiny pools waste capacity and create overhead.
- Why A is wrong: N/A
- Why B is wrong: Multi-tenant systems often need bulkheads.
- Why C is wrong: Different risk profiles justify isolation.
- Why D is wrong: Noise isolation is a core use case.

Q6)
- Correct: A
- Why correct: Saturated pools alongside idle pools indicate mis-sized partitions.
- Why A is wrong: N/A
- Why B is wrong: That suggests healthy balance.
- Why C is wrong: Cache hit ratio is unrelated.
- Why D is wrong: TLS timing is unrelated.

Q7)
- Correct: A
- Why correct: Bulkheads isolate capacity per component; backpressure limits overall load.
- Why A is wrong: N/A
- Why B is wrong: Bulkheads don’t replace backpressure.
- Why C is wrong: Backpressure doesn’t guarantee delivery.
- Why D is wrong: They solve different problems.

Q8)
- Correct: A
- Why correct: Per-tenant caps prevent a single tenant from dominating a pool.
- Why A is wrong: N/A
- Why B is wrong: Removing limits makes noisy neighbor problems worse.
- Why C is wrong: Disabling auth is unsafe and doesn’t isolate capacity.
- Why D is wrong: Merging pools removes isolation.

Q9)
- Correct: A
- Why correct: Over-partitioning wastes capacity and adds tuning overhead.
- Why A is wrong: N/A
- Why B is wrong: Bulkheads don’t guarantee extra availability.
- Why C is wrong: Retries and breakers are different mechanisms.
- Why D is wrong: Bulkheads don’t replace rate limiting.

Q10)
- Correct: A
- Why correct: Combining isolation, bounded time, and failure detection provides defense-in-depth.
- Why A is wrong: N/A
- Why B is wrong: Retries without timeouts amplify overload.
- Why C is wrong: A large shared pool increases blast radius.
- Why D is wrong: Monitoring is required to tune bulkheads.

## Common mistakes
- One global pool for all dependencies.
- No per-tenant isolation.
- Ignoring pool saturation signals.
