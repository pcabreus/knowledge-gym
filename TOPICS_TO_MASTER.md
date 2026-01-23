# Master Topic Backlog — Knowledge Gym

This is the master list of topics to dominate. Use it as the queue for creating canonical topics (one per file) and linking prerequisites/related topics.
Keep one canonical topic per concept. Avoid duplication by linking.

---

## A) Programming fundamentals & CS
- Data structures: arrays, linked lists (singly/doubly/circular), stacks, queues, deque, priority queue, hash tables, heaps, trees (BST), graphs
- Algorithms: sorting (bubble, insertion, merge, quick, heap sort), searching (linear, binary), recursion, complexity analysis (time/space), Big-O
- Graph traversal: BFS, DFS, shortest path basics (Dijkstra; when BFS is enough), adjacency matrix/list
- Concurrency basics: race conditions, deadlocks, starvation, critical sections
- Functional basics: pure functions, side effects, immutability

## B) Go (language + ecosystem)
- Go fundamentals: types, interfaces, errors, modules, generics (when to use)
- Concurrency: goroutines, channels, select, worker pools, fan-in/fan-out
- context.Context: cancellation, deadlines, propagation rules (what to put / not put)
- Testing: unit/integration, table-driven tests, mocks vs fakes, contract tests
- Profiling: pprof (CPU/heap/block), reading flamegraphs, interpreting results
- HTTP server/client: timeouts, retries, middleware, connection reuse
- DB access in Go: database/sql, sqlx, pgx (trade-offs)
- Logging: structured logging concepts, correlation IDs
- Performance: allocations, GC basics, pooling (when warranted)
- Reliability in Go: idempotency keys, retry backoff with jitter, rate limiting implementations

## C) API design & distributed systems
- REST API design: resources, methods, status codes, pagination, versioning, deprecations
- Idempotency semantics: GET vs PUT vs POST
- Authentication vs authorization
- JWT, Bearer tokens, headers, sessions
- Email verification: magic links vs OTP codes (abuse cases, UX)
- Rate limiting: token bucket/leaky bucket, 429 strategy, quotas
- Load balancing: L4 vs L7, sticky sessions, health checks
- Partitioning & scaling: sharding, partition keys, consistent hashing
- Caching: TTL, invalidation, cache-aside vs write-through, short-lived caching
- [Backpressure](topics/system-design/backpressure.md): bounded concurrency, queues, overload protection, graceful degradation
- Consistency models: strong vs eventual, read-after-write, monotonic reads
- CAP theorem (practical)

## D) Databases & data modeling
- SQL foundations: joins (inner/left), indexes, query planning
- PostgreSQL: EXPLAIN, index types (B-tree, GIN, GiST), JSONB, transactions, locks, deadlocks
- Normalization vs denormalization
- NoSQL: key-value access patterns, trade-offs vs SQL
- DynamoDB: single-table design, PK/SK, GSIs/LSIs, hot partitions, TTL, streams, consistency
- ACID properties; constraints vs enforcing in app
- Performance topics: read/write contention, connection pooling, slow query diagnosis

## E) Event-driven architecture & messaging
- Event-driven patterns (core)
- Kafka (Confluent): partitions, keys, ordering, consumer groups, offsets, delivery semantics
- SQS/SNS (incl. FIFO): ordering, dedup, visibility timeout, DLQ
- RabbitMQ vs Kafka (conceptual differences)
- Outbox pattern, CDC concepts
- Retry strategy: exponential backoff, jitter, retry budgets
- DLQ operations: replay process + RCA workflow

## F) Resilience & reliability engineering
- Idempotency (HTTP + message processing)
- Retries: what/when to retry, when not to retry
- Circuit breaker: states, thresholds, half-open behavior
- Timeouts: budgets, cascading failures
- Bulkheads, load shedding (conceptual)
- Error budgets; SLI/SLO vs SLA
- Incident management lifecycle; runbooks
- Incident-driven improvements

## G) AWS / Cloud architecture
- AWS Lambda: cold starts, timeouts, memory tuning, failure modes
- API Gateway: auth, throttling, limits/timeouts
- Step Functions: orchestration vs business logic, retries/compensation, idempotency in workflows
- AppSync (GraphQL): resolver patterns, N+1 pitfalls, authN/authZ
- IAM: least privilege, role separation, policy scoping
- EKS/Kubernetes (see section H)
- Short-lived caching patterns around third-party APIs

## H) Kubernetes / CI/CD / DevEx
- Docker: images, multi-stage builds, slimming, pinning versions
- Kubernetes/EKS: deployments, services, ingress basics, probes, autoscaling concepts
- GitHub Actions; Jenkins (concepts)
- CI vs CD (Delivery) vs CD (Deployment)
- Canary deployments; blue/green deployments
- Bazel monorepo: reproducible builds, caching, boundaries
- Terraform: plan/apply, modules, state + locking

## I) Observability
- Datadog APM/logs/traces (concepts + practice)
- RED & USE methodologies
- Structured logging: trace IDs, correlation IDs
- Metrics strategy: latency percentiles, error rate, saturation
- Alerting: actionable alerts vs noise

## J) Security & crypto
- Hashing vs encryption vs signing (differences)
- Password hashing: bcrypt vs argon2id (parameters, tuning)
- Key management basics: storage, rotation (conceptual)
- Webhook security: signature verification, replay protection
- OWASP basics (Top risks)

## K) Architecture & design patterns
- Domain-Driven Design (DDD): bounded contexts, aggregates (basics)
- Hexagonal architecture (ports & adapters)
- CQRS (command/query separation) + trade-offs
- Event sourcing: when it helps, when it hurts
- Anti-Corruption Layer (ACL): purpose, implementation, failure modes
- Translators/adapters patterns
- Microservices vs modular monolith (decision topic)

## L) Integrations & enrichment (from your CV/cases)
- Webhooks (ingestion patterns)
- Third-party data APIs (ZoomInfo/Clearbit-style enrichment)
- Data-use controls / compliance constraints
- Correlation windows, matching logic, deduplication strategy
- Short-lived caching for third-party APIs

## M) Interview mastery
### Behavioral / leadership themes
- Prioritization in ambiguous legacy + tight deadlines
- Disagreeing with product/design decisions (collaboration outcomes)
- Shipping fast with quality risk (risk management)
- Production incident leadership (incident narrative)
- Reducing tech debt without slowing delivery
- Leading without formal authority

### System design interview themes
- Designing resilient pipelines (retry/DLQ/idempotency)
- API + DB access pattern design
- Event-driven systems at scale
- Caching & rate limiting
- Observability strategy (SLI/SLO, dashboards)

## N) Your cases to convert into Case files
- Heimdall Step Functions orchestration + idempotency + retries/failover
- Heimdall failing writes into AppSync + retry/failover loop breaking
- Opportunity Actions architecture + third-party enrichment + caching + data-use policies
- Leads leak / incident (expand)
- DLQ replay / runbook stories (expand)

## O) Spec-driven development & AI-assisted engineering
- Spec → plan/steps → code workflow
- PR sizing for AI agents; reviewability
- Guardrails for code generation: tests, docs, security, edge cases
