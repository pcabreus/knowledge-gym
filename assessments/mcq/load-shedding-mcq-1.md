---
id: load-shedding-mcq-1
topic: ../topics/operations/load-shedding.md
kind: mcq
difficulty: medium
timebox_minutes: 20
created_at: 2026-01-23
updated_at: 2026-01-23
---

# Load shedding — MCQ 1

## Prompt
I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.
Q1) What is load shedding?
A) Intentionally rejecting or degrading work when demand exceeds safe capacity.
B) A database migration technique for zero downtime.
C) A security method for encrypting traffic at rest.
D) A scheduler that guarantees exactly-once execution.

Q2) Which statement best differentiates load shedding from backpressure?
A) Load shedding drops/degrades requests; backpressure slows or queues them.
B) Load shedding is only for databases; backpressure is only for HTTP.
C) Load shedding guarantees delivery; backpressure drops requests.
D) Load shedding is a monitoring feature; backpressure is logging.

Q3) Your API is overloaded and 95% CPU. What is the best immediate load-shedding response for non-critical endpoints?
A) Return 503 with `Retry-After` or serve a degraded response.
B) Increase timeouts so requests wait longer.
C) Spawn more goroutines and hope the CPU catches up.
D) Disable rate limits so all requests get through.

Q4) What is the primary trade-off of aggressive load shedding?
A) Stability improves but some users get rejected or degraded responses.
B) It guarantees exactly-once processing.
C) It removes the need for capacity planning.
D) It prevents all dependency failures.

Q5) When should you avoid load shedding and prefer async queueing?
A) When every request must be processed and delays are acceptable.
B) When traffic is spiky and short-lived.
C) When SLOs require low latency under overload.
D) When downstream capacity is limited.

Q6) Which statement best differentiates load shedding from a circuit breaker?
A) Shedding protects overall capacity; breakers stop calls to unhealthy dependencies.
B) Shedding is only client-side; breakers are only server-side.
C) Shedding improves request ordering; breakers improve message deduplication.
D) Shedding is a retry strategy; breakers are a caching strategy.

Q7) What is a common failure mode if clients retry without budgets after being shed?
A) Thundering herd that amplifies overload and prolongs incidents.
B) Automatic capacity scaling with no downsides.
C) Guaranteed eventual consistency.
D) Elimination of tail latency.

Q8) Which metric most directly indicates load-shedding activation?
A) Rejection rate with saturation indicators (CPU/pool usage).
B) Cache hit ratio.
C) Disk fragmentation percentage.
D) Schema migration count.

Q9) You must protect a high-priority endpoint while shedding low-priority traffic. Which policy is best?
A) Tiered admission control with per-endpoint budgets.
B) A single global queue for all endpoints.
C) Randomly reject all requests equally.
D) Disable authentication to speed up requests.

Q10) Which statement about load shedding and scaling is most accurate?
A) Shedding handles bursts; sustained overload still requires scaling.
B) Shedding replaces the need for scaling forever.
C) Shedding should be used only after scaling is complete.
D) Shedding is the same as horizontal autoscaling.

## Answer format (what “good” looks like)
- Correct letter per question
- Short justification focused on capacity, latency, and failure modes

## Answer key / Rubric
### Correct answer (for MCQ)
Q1)
- Correct: A
- Why correct: Load shedding is deliberate rejection or degradation under overload to protect system stability.
- Why A is wrong: N/A
- Why B is wrong: Migrations are unrelated to overload control.
- Why C is wrong: Encryption is unrelated to traffic control.
- Why D is wrong: Exactly-once execution is unrelated to shedding.

Q2)
- Correct: A
- Why correct: Backpressure slows or queues producers; load shedding drops or degrades work.
- Why A is wrong: N/A
- Why B is wrong: Both apply across many layers and protocols.
- Why C is wrong: Shedding doesn’t guarantee delivery.
- Why D is wrong: These are control mechanisms, not logging/monitoring.

Q3)
- Correct: A
- Why correct: Fast-fail with clear signaling protects capacity and preserves critical paths.
- Why A is wrong: N/A
- Why B is wrong: Longer waits increase contention and tail latency.
- Why C is wrong: Unbounded concurrency worsens overload.
- Why D is wrong: Removing limits increases load and failure.

Q4)
- Correct: A
- Why correct: The stability/latency benefit comes at the cost of rejected or degraded requests.
- Why A is wrong: N/A
- Why B is wrong: Shedding doesn’t provide exactly-once semantics.
- Why C is wrong: Capacity planning is still required.
- Why D is wrong: Shedding doesn’t prevent dependency failures.

Q5)
- Correct: A
- Why correct: If you must process every request, queueing with bounded concurrency is preferable.
- Why A is wrong: N/A
- Why B is wrong: Spiky traffic is a good fit for shedding.
- Why C is wrong: Low-latency SLOs often require shedding.
- Why D is wrong: Limited capacity is precisely why shedding can help.

Q6)
- Correct: A
- Why correct: Shedding manages capacity under overload; breakers stop calls to unhealthy dependencies.
- Why A is wrong: N/A
- Why B is wrong: Both can be server- or client-side depending on architecture.
- Why C is wrong: Ordering/dedup are unrelated.
- Why D is wrong: Shedding is not a retry strategy.

Q7)
- Correct: A
- Why correct: Unbounded retries reintroduce the same load and create a feedback loop.
- Why A is wrong: N/A
- Why B is wrong: Retrying doesn’t automatically scale capacity.
- Why C is wrong: Consistency is unrelated.
- Why D is wrong: Retrying often increases tail latency.

Q8)
- Correct: A
- Why correct: Shedding is visible as explicit rejections correlated with saturation signals.
- Why A is wrong: N/A
- Why B is wrong: Cache hit ratio doesn’t indicate shedding.
- Why C is wrong: Disk fragmentation is unrelated.
- Why D is wrong: Migrations are unrelated to overload control.

Q9)
- Correct: A
- Why correct: Tiered admission control preserves critical paths and sheds non-critical traffic first.
- Why A is wrong: N/A
- Why B is wrong: A single queue lets low-priority traffic crowd out critical work.
- Why C is wrong: Random shedding ignores priority requirements.
- Why D is wrong: Disabling auth increases risk and doesn’t reduce load.

Q10)
- Correct: A
- Why correct: Shedding is a short-term stability mechanism; sustained demand still requires capacity changes.
- Why A is wrong: N/A
- Why B is wrong: Shedding doesn’t replace scaling long term.
- Why C is wrong: Shedding is often needed before scaling catches up.
- Why D is wrong: Shedding and autoscaling solve different problems.

## Common mistakes
- Treating load shedding as “bigger queues.”
- Forgetting priority tiers.
- Allowing unbounded retries after rejection.
