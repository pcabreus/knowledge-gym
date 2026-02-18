I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.

Q1) Which statement best describes the CAP theorem trade-offs for a distributed system during a network partition?
A) You can have consistency and availability simultaneously during partitions
B) You must choose between consistency and availability while partitioned
C) Partitions are irrelevant if you use a distributed cache
D) CAP only applies to single-node databases
Correct: B
Why correct: Under partition, a distributed system must sacrifice either consistency or availability (CAP theorem). Systems choose AP or CP behavior.
Why A is wrong: You cannot guarantee both during partitions.
Why C is wrong: Distributed caches do not eliminate partitions.
Why D is wrong: CAP is a property of distributed systems, not single-node DBs.

Q2) What is the practical interpretation of "exactly-once" processing in distributed event systems?
A) Broker guarantees a message is delivered exactly once without additional logic
B) It is implemented by combining at-least-once delivery with idempotent consumer logic and transactional side effects
C) Exactly-once means no retries are ever needed
D) Exactly-once is achieved by making all services synchronous
Correct: B
Why correct: Exactly-once effect is realized by at-least-once delivery combined with idempotency and transactional behavior to avoid duplicate side effects.
Why A is wrong: Brokers alone rarely guarantee true exactly-once without consumer-side measures.
Why C is wrong: Retries are often part of robust systems; idempotency handles duplicates.
Why D is wrong: Synchronous designs don't inherently provide exactly-once across distributed boundaries.

Q3) Which mechanism helps prevent cascading failures from a slow downstream dependency?
A) Increasing timeouts to very high values
B) Circuit breaker patterns and bulkheads
C) Removing monitoring to reduce load
D) Using synchronous calls without timeouts
Correct: B
Why correct: Circuit breakers stop calling failing services and bulkheads isolate failures to prevent system-wide collapse.
Why A is wrong: Increasing timeouts can worsen resource exhaustion.
Why C is wrong: Removing monitoring worsens visibility and doesn't prevent failures.
Why D is wrong: Synchronous calls without timeouts risk hanging and cascading failures.

Q4) What is the difference between latency and throughput?
A) Latency measures aggregate operations per second; throughput measures single request duration
B) Latency is time per operation; throughput is number of operations per time unit
C) They are always inversely proportional and cannot be changed independently
D) Throughput is only relevant for batch jobs
Correct: B
Why correct: Latency is the time a single operation takes; throughput is how many operations can be processed per time unit.
Why A is wrong: It reverses definitions.
Why C is wrong: They can be traded off but are not strictly inversely proportional in all cases.
Why D is wrong: Throughput is relevant to many systems, not only batch jobs.

Q5) For leader-based replication, what is a common trade-off when choosing synchronous replication to all replicas?
A) Lower consistency, higher availability
B) Higher write latency, stronger consistency
C) Eliminates the need for leader election
D) Improves write throughput linearly with replicas
Correct: B
Why correct: Synchronous replication provides stronger consistency at the cost of higher write latency and potential availability issues if replicas are slow.
Why A is wrong: Synchronous replication increases consistency, not reduce it.
Why C is wrong: Leader election is still required for coordination.
Why D is wrong: Writes often get slower with more synchronous replicas, not faster.

Q6) What is the role of partitioning (sharding) in scaling databases?
A) Partitioning duplicates all data across nodes for redundancy
B) Partitioning splits data across nodes to increase throughput and storage capacity
C) Partitioning enforces global transactions cheaply
D) Partitioning reduces the need for backups
Correct: B
Why correct: Partitioning divides data so each node handles a subset, allowing parallelism and scale in throughput and storage.
Why A is wrong: Replication duplicates; partitioning splits.
Why C is wrong: Partitioning complicates global transactions; it doesn't make them cheaper.
Why D is wrong: Backups are still needed; partitioning doesn't remove that requirement.

Q7) Which consistency model allows reads to see eventual updates but may return stale data temporarily?
A) Strong consistency
B) Eventual consistency
C) Atomic consistency
D) Linearizability
Correct: B
Why correct: Eventual consistency permits temporary staleness but guarantees convergence over time.
Why A is wrong: Strong consistency guarantees read-after-write visibility.
Why C is wrong: "Atomic consistency" is not a standard consistency term here.
Why D is wrong: Linearizability is a strong consistency model, not eventual.

Q8) In distributed tracing, why is context propagation important?
A) It automatically fixes network partitions
B) It allows correlating spans across services to reason about latencies and root causes
C) It encrypts all inter-service traffic
D) It replaces metrics entirely
Correct: B
Why correct: Propagating trace context across calls enables building end-to-end traces to analyze latencies and failures.
Why A is wrong: Tracing doesn't fix partitions.
Why C is wrong: Tracing is about observability, not encryption (though traces may be secured separately).
Why D is wrong: Tracing complements metrics; it doesn't replace them.

Q9) Which algorithm is commonly used for leader election in distributed systems?
A) HTTP polling
B) Raft or Paxos consensus algorithms
C) Single-threaded scheduling
D) DNS round-robin
Correct: B
Why correct: Consensus algorithms like Raft or Paxos provide leader election and agreement primitives in distributed systems.
Why A is wrong: HTTP polling is not a robust consensus algorithm.
Why C is wrong: Single-threaded scheduling is a local runtime concept, not distributed leader election.
Why D is wrong: DNS round-robin is load distribution, not leader election.

Q10) What is an effective way to scale a processing pipeline for high throughput while maintaining reasonable latency?
A) Make every processing stage synchronous and single-threaded
B) Horizontally scale stateless workers, tune batch sizes, and ensure downstream systems can handle throughput
C) Centralize all processing in a single powerful node
D) Remove all timeouts to avoid aborted requests
Correct: B
Why correct: Horizontal scaling, batch tuning, and ensuring downstream capacity are core strategies to increase throughput while controlling latency.
Why A is wrong: Synchronous single-threaded stages limit throughput.
Why C is wrong: Centralizing causes bottlenecks and single point of failure.
Why D is wrong: Removing timeouts risks resource exhaustion and cascading failures.
