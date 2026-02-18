I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.

Q1) Which of the following best describes an event in an event-driven architecture?
A) A mutable record stored in a database table
B) An immutable message representing something that happened in the system
C) A direct HTTP request between services
D) A scheduled cron job entry
Correct: B
Why correct: Events are immutable messages representing facts (e.g., OrderCreated) that can be published and consumed asynchronously.
Why A is wrong: Events are normally immutable messages; database records may be mutable representations.
Why C is wrong: HTTP requests are synchronous calls, not the canonical event message pattern.
Why D is wrong: Cron jobs are scheduled tasks, not domain events.

Q2) In which scenario is a Topic (pub/sub) preferable to a Queue (point-to-point)?
A) When only one consumer should process each message
B) When multiple independent services need to react to the same event
C) When you need strict FIFO across unrelated consumers
D) When message ordering is irrelevant
Correct: B
Why correct: Topics allow multiple subscribers to receive the same event, enabling different services to react independently.
Why A is wrong: A Queue is used when each message should be processed by one consumer.
Why C is wrong: Strict FIFO across unrelated consumers isn't guaranteed by most pub/sub systems; queues are for single-consumer processing.
Why D is wrong: This option doesn't justify choosing a topic.

Q3) What is the main reason to use an append-only event log (e.g., Kafka) in a system?
A) To allow in-place updates of historical events
B) To enable replayability and maintain an ordered history of events
C) To speed up single-row transactional updates in a DB
D) To replace the need for backups
Correct: B
Why correct: Append-only logs serve as ordered, durable histories allowing consumers to replay events and rebuild state.
Why A is wrong: Append-only logs are not for in-place updates; they append new events.
Why C is wrong: They are not primarily for speeding DB updates; they complement databases for event-driven flows.
Why D is wrong: Logs are not a replacement for backups; they are a different persistence model.

Q4) Which technique prevents double-processing of events under at-least-once delivery?
A) Dropping duplicate messages at the broker
B) Making consumers idempotent using deduplication keys and an idempotency table
C) Using at-most-once delivery only
D) Only processing messages during off-peak hours
Correct: B
Why correct: Idempotent consumers check deduplication keys or persisted markers before applying side effects, enabling safe retries.
Why A is wrong: Brokers may help with deduplication but relying solely on brokers is fragile; consumer-side idempotency is the robust approach.
Why C is wrong: At-most-once avoids duplicates but sacrifices reliability; it's often unacceptable.
Why D is wrong: Timing does not prevent duplicates.

Q5) What is a Dead-Letter Queue (DLQ) used for?
A) To store successfully processed messages for auditing
B) To hold messages that repeatedly fail processing so they can be inspected and handled separately
C) To automatically delete messages older than retention
D) To balance load across consumers
Correct: B
Why correct: DLQs collect messages that fail processing after retries, enabling manual inspection or special handling.
Why A is wrong: DLQs are for failed messages, not successful audit logs.
Why C is wrong: DLQs aren't a retention deletion mechanism.
Why D is wrong: DLQs do not balance load; they isolate problematic messages.

Q6) Which pattern helps ensure transactional consistency when publishing events alongside DB updates?
A) Two-phase commit between DB and broker
B) Outbox pattern (write event to DB table within transaction, then publish reliably)
C) Publish event first then update DB without coordination
D) Using eventual consistency without persistence
Correct: B
Why correct: The Outbox pattern writes the event within the same DB transaction as the state change, then a reliable publisher forwards outbox entries to the broker, avoiding dual-write issues.
Why A is wrong: Two-phase commit across heterogeneous systems is complex and rarely used for scale/availability reasons.
Why C is wrong: Publishing before DB commit risks losing consistency if the DB transaction fails.
Why D is wrong: While eventual consistency is used, it doesn't solve atomic DB+event persistence without patterns like Outbox.

Q7) What does backpressure aim to solve in event-driven systems?
A) Increase the raw throughput indefinitely
B) Slow down or shed incoming load when consumers or downstream systems cannot keep up
C) Guarantee exactly-once delivery without idempotency
D) Replace monitoring and alerting
Correct: B
Why correct: Backpressure mechanisms throttle or shed load to prevent overwhelming consumers and downstream systems.
Why A is wrong: Backpressure constrains throughput to safe levels, not increase it indefinitely.
Why C is wrong: Backpressure doesn't guarantee delivery semantics; idempotency handles duplicates.
Why D is wrong: Backpressure complements monitoring but doesn't replace it.

Q8) Which is true about replaying events from an append-only log?
A) Replaying always needs no changes to consumers
B) Replaying is useful for rebuilding state and debugging, but consumers must handle idempotency and ordering considerations
C) Replay is unnecessary if you have a stateless system
D) Replay invalidates previous events
Correct: B
Why correct: Replayability allows rebuilding state, but consumers must be designed for idempotency and correct ordering to avoid inconsistent state.
Why A is wrong: Consumers often need to be idempotent and may need version compatibility handling for event schemas.
Why C is wrong: Even stateless consumers sometimes need replay for reprocessing or analytics; replay has clear uses.
Why D is wrong: Replay does not invalidate events; it reprocesses them.

Q9) What trade-off does choosing Kafka (log-based) over SQS (queue-based) typically imply?
A) Kafka offers simpler single-writer semantics while SQS provides global ordering
B) Kafka provides high-throughput ordered log retention suitable for replay; SQS is simpler for point-to-point queues with simpler operational model
C) Kafka enforces exactly-once delivery by default; SQS does not
D) Kafka removes the need for consumer-group coordination
Correct: B
Why correct: Kafka is a durable, ordered log that supports high throughput and replay; SQS is a managed queue service simpler for point-to-point processing.
Why A is wrong: Kafka is log-based, not necessarily simpler for single-writer semantics.
Why C is wrong: Exactly-once semantics require careful setup; Kafka doesn't magically provide exactly-once in all cases.
Why D is wrong: Kafka uses consumer groups to coordinate consumption; it doesn't remove coordination needs.

Q10) Which is the most practical approach for handling schema evolution of events?
A) Never change event schemas once deployed
B) Use backward/forward compatible schema changes with versioning and compatibility checks (e.g., schemas, consumers tolerant to missing fields)
C) Force all consumers to update simultaneously when schema changes
D) Store events as untyped binary blobs without schemas
Correct: B
Why correct: Evolving schemas with backward/forward compatibility and versioning ensures new and old consumers can coexist; compatibility tooling helps enforce safe changes.
Why A is wrong: In practice, schemas must evolve; forbidding changes is unrealistic.
Why C is wrong: Simultaneous updates are impractical at scale.
Why D is wrong: Untyped blobs lose observability and compatibility guarantees.
