---
id: choosing-right-database
title: "Choosing the Right Database"
type: playbook
category: decision-making
difficulty: hard
importance: 95
tags: [database, system-design, architecture, trade-offs, decision-making]
created_at: 2026-02-12
updated_at: 2026-02-12
---

# Choosing the Right Database

## Situation
**When this applies:**
- Starting new project or service
- Migrating from existing database
- Scaling beyond current database capabilities
- Architecting new feature with specific data requirements

**Common manifestations:**
- "Should we use PostgreSQL or DynamoDB?"
- "We need full-text search—add to DB or use Elasticsearch?"
- "Our MySQL instance can't handle the write load"
- "Do we need a graph database for this social feature?"

## Why this matters
- Database choice affects scalability, latency, cost, and operational complexity
- Wrong choice causes painful migrations later
- No "best" database—only trade-offs
- Critical interview topic: demonstrates systems thinking, trade-off analysis, experience
- Senior engineers choose based on requirements, not hype

## My Process (Step-by-step)

### 1. Define access patterns and requirements
- **What:** List how data will be read and written
- **Why:** Database optimizes for specific patterns; start with usage, not storage model
- **How:** Document: read patterns, write patterns, query types, consistency needs
- **Example:**
  - **Reads:** 95% reads by `user_id`, 5% by `email`; p95 <100ms
  - **Writes:** 1000 inserts/sec, batch updates hourly
  - **Queries:** No complex joins, no full-text search, no analytics
  - **Consistency:** Eventual OK for reads, strong for writes

### 2. Quantify scale and growth
- **What:** Estimate data size, throughput, and growth rate
- **Why:** Different databases scale differently; small vs large changes the answer
- **How:** Calculate: rows/documents, storage size, QPS (read + write), growth per month
- **Example:**
  - **Current:** 10M users, 100M orders, 500GB data
  - **Throughput:** 5k reads/sec, 500 writes/sec
  - **Growth:** 20% per month (2x in 4 months)
  - **Forecast:** 200M orders, 2TB data in 12 months

### 3. Identify constraints
- **What:** List hard requirements that eliminate options
- **Why:** Constraints narrow decision space
- **How:** Consider: budget, team expertise, compliance, latency SLA, availability SLA
- **Example:**
  - **Budget:** <$5k/month
  - **Team:** Strong PostgreSQL experience, zero NoSQL experience
  - **Compliance:** GDPR (data in EU)
  - **Latency:** p99 <200ms
  - **Availability:** 99.9% (not 99.99%)

### 4. Map requirements to database characteristics
- **What:** Match access patterns to database strengths
- **Why:** Each database optimized for specific use cases
- **How:** Consider:
  - **Relational (PostgreSQL, MySQL):** Complex queries, transactions, strong consistency, schema enforcement
  - **Key-value (DynamoDB, Redis):** Simple lookups by key, horizontal scaling, eventual consistency
  - **Document (MongoDB):** Flexible schema, nested data, evolving structure
  - **Search (Elasticsearch):** Full-text search, analytics, fuzzy matching
  - **Graph (Neo4j):** Relationship traversal, social networks, recommendations
- **Example:**
  - **Requirement:** Mostly lookups by `user_id` (key-value pattern) + strong consistency  + moderate scale
  - **Match:** PostgreSQL (handles 5k QPS easily, strong consistency, team knows it) OR DynamoDB (scales better but eventual consistency)

### 5. Evaluate trade-offs explicitly
- **What:** List pros/cons of top 2-3 candidates
- **Why:** Makes reasoning transparent and enables informed choice
- **How:** Compare: scalability, consistency, query flexibility, operational complexity, cost
- **Example:**
  ```
  **Option A: PostgreSQL**
  - ✅ Strong consistency, complex queries, team expertise
  - ✅ Cheap ($500/month RDS)
  - ❌ Vertical scaling limits (~50k QPS), sharding is complex
  
  **Option B: DynamoDB**
  - ✅ Horizontal scaling (unlimited QPS), managed service
  - ✅ p99 <10ms at any scale
  - ❌ Eventual consistency, limited query patterns, learning curve
  - ❌ More expensive (~$2k/month at scale)
  
  **Option C: MongoDB**
  - ✅ Flexible schema, horizontal scaling
  - ❌ Team has no experience, operational complexity, less mature than Postgres
  ```

### 6. Choose optimizing for primary constraint
- **What:** Select database that best satisfies most important requirement
- **Why:** Can't optimize for everything; pick what matters most
- **How:** State: "I optimize for X over Y because [business/technical reason]"
- **Example:**
  - **Decision:** PostgreSQL
  - **Rationale:** "Optimizing for team velocity (Postgres expertise) and cost (<$500/month) over scale. Current 5k QPS is <10% of Postgres limit. We'll revisit if we hit 30k QPS (18 months at 20% growth)."

### 7. Plan migration escape hatch
- **What:** Design abstraction layer in case you need to switch later
- **Why:** Requirements change; make migration possible (not easy, just possible)
- **How:** Use repository pattern, avoid DB-specific features initially, design for data portability
- **Example:**
  - **Abstraction:** `UserRepository` interface, Postgres implementation
  - **Future-proof:** Avoid Postgres-specific features (JSONB, arrays) until proven necessary
  - **Export:** Design API to export data in portable format (JSON/CSV)

### 8. Validate with prototype
- **What:** Build spike to test critical path
- **Why:** Theory differs from reality; prototype reveals unknowns
- **How:** Timebox 2-3 days, test: write throughput, read latency, query complexity
- **Example:**
  - **Spike:** Load 10M rows into Postgres, test queries at 10k QPS
  - **Result:** p95 = 50ms (✅), p99 = 150ms (✅), write throughput 2k/sec (✅)
  - **Validated:** Postgres handles current + 2x growth

## Success Signals
**You know it's working when:**
- [ ] Database meets latency/throughput requirements
- [ ] Operational complexity manageable by team
- [ ] Cost within budget
- [ ] Access patterns supported efficiently
- [ ] Migration path exists if requirements change

## Common Pitfalls (and how to avoid them)

### Pitfall 1: Choosing based on hype, not requirements
- **Why it happens:** FOMO, resume-driven development
- **Impact:** Operational complexity, team learning curve, cost
- **Instead do:** Start with access patterns and constraints

### Pitfall 2: Premature optimization for scale
- **Why it happens:** "We might need billions of rows"
- **Impact:** Over-engineered, expensive, complex for small scale
- **Instead do:** Choose for current + 2x growth; revisit when needed

### Pitfall 3: Ignoring team expertise
- **Why it happens:** Assuming "we'll learn it"
- **Impact:** Slow delivery, operational issues, downtime
- **Instead do:** Factor in learning curve; PostgreSQL you know >> MongoDB you don't

### Pitfall 4: Not validating with prototype
- **Why it happens:** Trust in documentation
- **Impact:** Discover limitations after migration (too late)
- **Instead do:** Build spike to validate critical requirements

## Real-world Example
**Context:** Building analytics dashboard for 100M events/day; needed to choose database.

**What happened:** Team debated PostgreSQL (familiar) vs ClickHouse (analytics-optimized) vs Elasticsearch (search-optimized).

**Process applied:**
1. **Access patterns:**
   - **Reads:** Aggregations (COUNT, SUM by date/user), no full-text search
   - **Writes:** 100M events/day (1.2k writes/sec), append-only
   - **Queries:** Time-range filters, group by, simple JOINs
   - **Consistency:** Eventual OK (analytics, not transactional)
2. **Scale:**
   - **Current:** 100M events/day = 3TB/month
   - **Growth:** 50% per month
   - **Forecast:** 1B events/day in 6 months
3. **Constraints:**
   - **Budget:** <$3k/month
   - **Team:** Strong Postgres, no ClickHouse or ES experience
   - **Latency:** p95 <3s for dashboard queries (not real-time)
   - **Retention:** 90 days (older data archived)
4. **Mapped:**
   - **PostgreSQL:** Can handle writes, but aggregations slow on billions of rows
   - **ClickHouse:** Optimized for analytics, columnar storage, fast aggregations
   - **Elasticsearch:** Over-kill (we don't need search), expensive
5. **Trade-offs:**
   ```
   **Postgres:**
   - ✅ Team knows it, cheap, good for 1-10M rows/day
   - ❌ Slow aggregations on billions of rows, not built for analytics
   
   **ClickHouse:**
   - ✅ Fast aggregations (100x faster), columnar storage, compression
   - ✅ Handles billions of rows easily
   - ❌ Team learning curve, operational complexity
   
   **Elasticsearch:**
   - ❌ Expensive ($5k/month), we don't need search features
   ```
6. **Decision:** ClickHouse
   - **Rationale:** "Optimizing for query performance at scale (100M → 1B events/day). Willing to invest 2 weeks learning curve because Postgres can't handle forecast scale. Cost $2k/month (within budget)."
7. **Escape hatch:**
   - **Abstraction:** `EventsRepository` interface
   - **Export:** Daily exports to S3 (can reload into different DB)
8. **Validated:**
   - **Spike (3 days):** Loaded 500M events, ran dashboard queries
   - **Result:** Queries <500ms (vs 10s in Postgres), writes 5k/sec (✅), compression 10:1

**Outcome:** ClickHouse handled 1B events/day with <1s query latency, $1.8k/month cost, team ramped in 2 weeks.

## Interview Tips
### How to talk about this
- **Framework to use:** "Access patterns → scale → constraints → map to DB characteristics → trade-offs → choose → escape hatch → validate"
- **Key points to emphasize:** Requirements-driven, trade-off analysis, team context, validation
- **Language that works:** "defined access patterns," "quantified scale," "mapped to," "optimized for X over Y," "validated with prototype"

### Sample STAR response outline
- **Situation:** "Needed to choose database for [use case: analytics, social, transactions] handling [scale]"
- **Task:** "Required [latency/throughput/consistency] within [budget/team/compliance] constraints"
- **Action:** "Defined access patterns, quantified scale/growth, evaluated [DB options], made trade-offs explicit (optimized for X over Y), validated with prototype"
- **Result:** "Chose [DB], met requirements ([metric]), cost [budget], team ramped in [time], system scaled to [growth]"

### Red flags to avoid
- Don't claim "X is always better than Y" (context matters)
- Don't skip access patterns ("we chose NoSQL because it's fast")
- Don't ignore team expertise (learning curve is real cost)
- Don't forget validation (document assumptions can be wrong)

## Self-assessment
- [ ] I can describe database choice I made and why
- [ ] I can list the access patterns that drove the decision
- [ ] I can explain the trade-offs of top 2-3 options
- [ ] I can state what I optimized for (and what I sacrificed)
- [ ] I can describe the validation I did (prototype or production results)

## Related Playbooks
- [Making Trade-off Decisions](making-trade-offs.md)
- [Handling Ambiguity](handling-ambiguity.md)
- [Applying Deep Technical Knowledge](../technical/applying-deep-knowledge.md)
- [Evaluating Project Success](../leadership/evaluating-project-success.md)

## Related Topics (knowledge base)
- [PostgreSQL](../../topics/databases/postgresql.md)
- [DynamoDB](../../topics/databases/dynamodb.md)
- [Redis](../../topics/databases/redis.md)
- [MongoDB Documents](../../topics/databases/mongodb-documents.md)
- [Cassandra](../../topics/databases/cassandra.md)
- [NoSQL Access Patterns](../../topics/databases/nosql-access-patterns.md)
- [Consistency Models](../../topics/databases/consistency-models.md)
- [CAP Theorem](../../topics/system-design/cap-theorem.md)
- [PACELC](../../topics/system-design/pacelc.md)

## Notes / Refinements
- Default to relational (Postgres/MySQL) unless specific reason not to
- NoSQL when: simple access patterns, massive scale, flexible schema, or key-value lookups
- Multi-database is OK: Postgres for transactions, Redis for cache, Elasticsearch for search
- Managed > self-hosted (unless cost or compliance prevents)
- Common mistake: choosing for imagined future scale instead of actual current + 2x
