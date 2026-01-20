
# Topics Index

## Database systems
### PostgreSQL
- [PostgreSQL](databases/postgresql.md)
- [PostgreSQL MVCC](databases/postgresql-mvcc.md)
- [PostgreSQL vacuum & autovacuum](databases/postgresql-vacuum-autovacuum.md)
- [PostgreSQL bloat](databases/postgresql-bloat.md)
- [PostgreSQL isolation levels](databases/postgresql-isolation-levels.md)
- [PostgreSQL statistics & ANALYZE](databases/postgresql-stats-and-analyze.md)
- [JSONB](databases/jsonb.md)
- [GIN index](databases/gin-index.md)

### DynamoDB
- [DynamoDB](databases/dynamodb.md)
- [DynamoDB single-table design](databases/dynamodb-single-table.md)
- [DynamoDB multi-table design](databases/dynamodb-multi-table.md)
- [DynamoDB keys (PK/SK)](databases/dynamodb-keys.md)
- [DynamoDB GSI/LSI](databases/dynamodb-gsi-lsi.md)
- [DynamoDB hot partitions](databases/dynamodb-hot-partitions.md)
- [DynamoDB TTL](databases/dynamodb-ttl.md)
- [DynamoDB Streams](databases/dynamodb-streams.md)
- [DynamoDB consistency](databases/dynamodb-consistency.md)

### MongoDB
- [MongoDB documents](databases/mongodb-documents.md)

### MySQL
- [MySQL](databases/mysql.md)

### Redis
- [Redis](databases/redis.md)
- [Redis eviction policies](databases/redis-eviction-policies.md)
- [Redis data types](databases/redis-data-types.md)
- [Redis Pub/Sub](databases/redis-pub-sub.md)
- [Redis scaling (scale up and out)](databases/redis-scaling.md)
- [Redis Streams](databases/redis-streams.md)
- [Redis persistence (RDB/AOF)](databases/redis-persistence.md)
- [Redis replication & Sentinel](databases/redis-replication-sentinel.md)

### SQLite
- [SQLite](databases/sqlite.md)

### Cassandra
- [Cassandra](databases/cassandra.md)

## Cross-cutting database concepts
### Indexing & query planning
- [Index](databases/index.md)
- [B-Tree index](databases/btree-index.md)
- [GiST](databases/gist-index.md)
- [Selectivity](databases/selectivity.md)
- [EXPLAIN](databases/explain.md)
- [Query planning](databases/query-planning.md)
- [Database statistics](databases/database-statistics.md)

### SQL fundamentals
- [Relational model basics](databases/relational-model-basics.md)
- [Data modeling basics](databases/data-modeling-basics.md)
- [SQL foundations](databases/sql-foundations.md)
- [SQL joins (inner/left)](databases/sql-joins.md)

### Transactions & concurrency
- [Transactions](databases/transactions.md)
- [Locks](databases/locks.md)
- [Lock-based concurrency control](databases/lock-based-concurrency-control.md)
- [Optimistic concurrency control](databases/optimistic-concurrency-control.md)
- [Deadlocks](databases/deadlocks.md)
- [ACID properties](databases/acid-properties.md)
- [Isolation levels (general)](databases/isolation-levels.md)
- [Read/write contention](databases/read-write-contention.md)

### Data modeling
- [Normalization](databases/normalization.md)
- [Denormalization](databases/denormalization.md)
- [Constraints vs app-level enforcement](databases/constraints-vs-app.md)
- [Data integrity basics](databases/data-integrity-basics.md)

### NoSQL & distributed patterns
- [NoSQL access patterns](databases/nosql-access-patterns.md)
- [Key-value data modeling](databases/key-value-data-modeling.md)
- [Document databases (overview)](databases/document-databases.md)
- [Hot partitions](databases/hot-partitions.md)
- [Consistency models](databases/consistency-models.md)
- [Change data capture (CDC)](databases/change-data-capture.md)
- [TTL (Time-to-Live)](databases/ttl.md)

### Operations & performance
- [Database client basics](databases/database-client-basics.md)
- [Connection pooling](databases/connection-pooling.md)
- [Slow query diagnosis](databases/slow-query-diagnosis.md)
- [Storage compaction](databases/storage-compaction.md)
- [Storage fundamentals](databases/storage-fundamentals.md)

## System design
- [Requirements basics](system-design/requirements-basics.md)
- [Caching fundamentals](system-design/caching-fundamentals.md)
- [Caching strategy](system-design/caching-strategy.md)
- [Performance basics](system-design/performance-basics.md)
- [Distributed systems basics](system-design/distributed-systems-basics.md)
- [Partitioning & sharding](system-design/partitioning-and-sharding.md)
- [CAP theorem (practical)](system-design/cap-theorem.md)
- [PACELC](system-design/pacelc.md)
- [API design basics](system-design/api-design-basics.md)
- [API validation](system-design/api-validation.md)

## Architecture
- [Messaging basics](architecture/messaging-basics.md)
- [Request-response architecture](architecture/request-response.md)
- [Event-driven basics](architecture/event-driven-basics.md)
- [Dual-write pattern](architecture/dual-write-pattern.md)
- [Outbox pattern](architecture/outbox-pattern.md)
- [Kafka](architecture/kafka.md)
- [RabbitMQ vs Kafka](architecture/rabbitmq-vs-kafka.md)

## Operations
- [Scheduled cleanup jobs](operations/scheduled-cleanup-jobs.md)
- [Maintenance windows](operations/maintenance-windows.md)
- [Database proxies](operations/database-proxies.md)
- [Batch ETL](operations/batch-etl.md)
- [Operational planning](operations/operational-planning.md)
- [Networking basics](operations/networking-basics.md)
- [Reliability basics](operations/reliability-basics.md)
- [Availability basics](operations/availability-basics.md)
- [Observability basics](operations/observability-basics.md)
- [Capacity planning](operations/capacity-planning.md)
- [Incident management basics](operations/incident-management-basics.md)
- [Infrastructure as Code (IaC)](operations/infrastructure-as-code.md)
- [Terraform](operations/terraform.md)
- [AWS CloudFormation](operations/aws-cloudformation.md)
- [Blue/green deployments](operations/blue-green-deployments.md)
- [Canary deployments](operations/canary-deployments.md)
- [Rolling deployments](operations/rolling-deployments.md)
- [Feature flags](operations/feature-flags.md)

## Quality & Engineering Practices
- [Software Quality Assurance (SQA)](quality/software-quality-assurance.md)
- [Testing fundamentals](quality/testing-fundamentals.md)
- [Testing pyramid](quality/testing-pyramid.md)
- [Code quality](quality/code-quality.md)
- [Quality gates (CI/CD)](quality/quality-gates.md)
- [Clean code](quality/clean-code.md)
- [SOLID principles](quality/solid-principles.md)
- [Design patterns (overview)](quality/design-patterns.md)
- [Refactoring techniques](quality/refactoring-techniques.md)
- [CI/CD basics](quality/ci-cd-basics.md)

## (Add more categories according to `docs/04-taxonomy.md`)
