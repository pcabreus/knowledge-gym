---
id: scaling-reads
title: "Scaling Reads"
type: pattern
status: learning
importance: 85
difficulty: medium
tags: [system-design, scalability, replication, caching, database]
canonical: true
aliases: [read-scaling, read-replicas]
created_at: 2026-02-12
updated_at: 2026-02-12
---

# Scaling Reads

## TL;DR (BLUF)
- Scale read capacity through caching, read replicas, or denormalization.
- Use caching for highest performance; replicas for strong consistency needs; denormalization when joins are too expensive.
- Key trade-off: eventual consistency vs synchronous replication cost.

## Definition
**What it is:** Techniques to handle increasing read traffic without overloading a primary database or service.  
**Key terms:** read replica, replication lag, cache hit ratio, denormalization, eventual consistency, leader-follower replication.

## Why it matters
- Most applications are read-heavy (90%+ reads vs writes).
- Read bottlenecks cause user-facing latency and can cascade to writes.
- Incorrectly scaled reads lead to stale data bugs or wasted infrastructure.
- Critical interview topic: demonstrates understanding of CAP, caching, and replication trade-offs.

## Scope & Non-goals
**In scope:**
- Caching strategies (application, CDN, database query cache)
- Read replicas (async replication, lag management)
- Denormalization for read optimization
- Read-write splitting patterns

**Out of scope / NOT solved by this:**
- Write scaling (see [Partitioning & Sharding](partitioning-and-sharding.md))
- Strong consistency across replicas (writes still go to primary)
- Cross-region consistency guarantees

## Mental model / Intuition
Think of a library: instead of everyone waiting in line at the main desk (primary database), you create copies of popular books (caching), open satellite branches with slightly delayed catalog updates (read replicas), or prepare summary sheets (denormalization). Each approach trades freshness for capacity.

## Decision rules (When to use / When not to use)
### Use it when
- Read traffic >> write traffic (typical ratio: 10:1 or higher)
- Users can tolerate seconds to minutes of staleness
- Read queries are expensive (complex joins, aggregations)
- Primary database CPU/memory is saturated by reads
- You need multi-region read performance

### Avoid it when
- Reads require strong consistency (e.g., financial balance checks, inventory counts)
- Write volume is the bottleneck (replicas can't help writes)
- Application logic can't handle stale reads
- Team lacks monitoring for replication lag

## How I would use it (practical)
- **Context:** E-commerce product catalog with 1000 req/s reads, 10 req/s writes
- **Steps:**
  1. Add application-level cache (Redis/Memcached) for hot products → reduces DB load 80%
  2. Route user-facing reads to read replicas; keep admin writes on primary
  3. Monitor replication lag; alert if >5 seconds
  4. Denormalize product reviews count into product table to avoid join
- **What success looks like:** Primary CPU <50%, p95 read latency <100ms, cache hit ratio >85%, replication lag <2s

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Linear read capacity scaling (add more replicas)
  - Offloads primary database for write performance
  - Geographic distribution for low latency
  - Cost-effective (replicas cheaper than scaling primary)
- **Cons / Risks:**
  - Eventual consistency: users may see stale data
  - Replication lag during high write load or network issues
  - Complexity: application must route reads vs writes
  - Replica failover increases operational burden

### Alternatives
- **Vertical scaling (bigger primary):** When consistency matters more than cost
- **CQRS with event sourcing:** When reads and writes have fundamentally different models
- **Federated databases:** When data naturally partitions (e.g., by tenant)

### How to choose
1. Start with caching (fastest wins, minimal complexity)
2. Add read replicas if cache misses are still overloading primary
3. Use denormalization for specific expensive queries
4. Consider CQRS only when read/write models diverge significantly

## Failure modes & Pitfalls
1. **Unbounded replication lag**: Network partition or bulk write causes replica to fall minutes/hours behind → users see very stale data → customer complaints
   - **Mitigation**: Monitor lag; circuit-break to primary if lag >threshold; use synchronous replication for critical reads

2. **Read-after-write inconsistency**: User updates profile, immediately refreshes page, sees old data (read from lagging replica)
   - **Mitigation**: Sticky sessions (same user → same replica), or route authenticated user reads to primary for N seconds after write

3. **Cache stampede on invalidation**: Popular item cache expires under heavy load → all requests hit database simultaneously → overload
   - **Mitigation**: Stale-while-revalidate pattern, probabilistic early expiration

4. **Split-brain during failover**: Primary fails, two replicas both attempt promotion → writes diverge
   - **Mitigation**: Use consensus-based leader election (e.g., PostgreSQL Patroni, MySQL Group Replication)

5. **Denormalization drift**: Denormalized data gets out of sync with source of truth due to bug in update logic
   - **Mitigation**: Reconciliation jobs, idempotent updates, single source of truth with eventual consistency guarantees

## Observability (How to detect issues)
- **Metrics:**
  - Replication lag (seconds behind primary)
  - Cache hit ratio (target >80%)
  - Read QPS per replica
  - Primary vs replica traffic ratio
  - Read latency p95/p99 per replica
- **Logs:**
  - Replication errors or disconnects
  - Cache invalidation patterns
  - Read-after-write staleness complaints (user-facing)
- **Traces:**
  - Span showing cache miss → replica query → primary fallback
- **Alerts:**
  - Replication lag >10 seconds
  - Cache hit ratio <70%
  - Replica unhealthy or unreachable
  - Read latency p95 >500ms

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Choose replication mode: async (default), sync (high consistency need), semi-sync (hybrid)
  - [ ] Configure read-write splitting in connection pool or proxy (e.g., ProxySQL, PgBouncer)
  - [ ] Set up replication lag monitoring and alerting
  - [ ] Implement cache-aside pattern with TTL matching staleness tolerance
  - [ ] Add replica health checks and automatic failover
  - [ ] Document which queries go to replica vs primary
  - [ ] Test failover scenarios (replica failure, primary failure)

- **Security / Compliance notes**
  - Encrypt replication traffic (TLS)
  - Ensure replicas have same access controls as primary
  - PII/GDPR: deletes must propagate to replicas

- **Performance notes**
  - Replicas can lag under write bursts; monitor and scale accordingly
  - Use connection pooling to avoid overwhelming replicas
  - Index replicas same as primary (they serve same queries)

- **Operational notes**
  - Schedule backups from replicas (avoid primary load)
  - Promote replica to primary during maintenance
  - Keep at least 2 replicas for availability during maintenance

## Mini example (if applicable)
```python
# Read-write splitting with manual routing
class UserService:
    def __init__(self, primary_db, replica_db, cache):
        self.primary = primary_db
        self.replica = replica_db  
        self.cache = cache
    
    def get_user(self, user_id):
        # Try cache first
        cached = self.cache.get(f"user:{user_id}")
        if cached:
            return cached
        
        # Read from replica (eventual consistency OK)
        user = self.replica.query("SELECT * FROM users WHERE id = ?", user_id)
        self.cache.set(f"user:{user_id}", user, ttl=300)
        return user
    
    def update_user(self, user_id, data):
        # Writes go to primary
        self.primary.execute("UPDATE users SET ... WHERE id = ?", data, user_id)
        
        # Invalidate cache to avoid stale reads
        self.cache.delete(f"user:{user_id}")
        
        # Optional: route this user's reads to primary for 10s
        self.cache.set(f"route_primary:{user_id}", True, ttl=10)
```

## Common anti-patterns
- **Anti-pattern:** Reading sensitive data (account balance, inventory count) from replica without checking lag
  - **Why it's bad:** User sees stale balance, makes bad decision (e.g., thinks they have money they don't)
  - **Better approach:** Route consistency-critical reads to primary or use synchronous replication

- **Anti-pattern:** Adding 10 read replicas without caching
  - **Why it's bad:** Expensive infrastructure solving a problem caching would handle for 1/10th the cost
  - **Better approach:** Start with caching; add replicas only if cache hit ratio is high but load remains

- **Anti-pattern:** Denormalizing without a reconciliation strategy
  - **Why it's bad:** Data drift accumulates over time, silently showing wrong data
  - **Better approach:** Run daily reconciliation jobs comparing denormalized values with source of truth

## Interview readiness
### "Explain it like I'm teaching"
Scaling reads means handling more read traffic without overloading your primary database. The best approach is caching: store frequently accessed data in memory (Redis/Memcached) so you don't hit the database at all. If caching isn't enough or you need fresher data, use read replicas—copies of your database that handle read queries while writes still go to the primary. The trade-off is eventual consistency: replicas lag behind the primary by seconds, so users might see slightly stale data. You route read queries to replicas and write queries to the primary, monitoring replication lag to ensure it stays acceptable. For specific expensive queries, you can denormalize data—duplicate it in a way that makes reads fast, like storing a precomputed count instead of doing a join every time.

### Trap questions (with answers)
1) **Q:** Do read replicas help with write bottlenecks?
   - **A:** No. Replicas still receive all writes from the primary (via replication), so writes don't scale. They only offload read queries. For write scaling, you need sharding or partitioning.

2) **Q:** Why not just add more read replicas instead of caching?
   - **A:** Replicas are more expensive (full database instances) and still hit disk/CPU for every query. Caching serves from memory and can handle 10-100x more traffic per dollar. Use caching first, replicas for consistency needs caching can't meet.

3) **Q:** How do you prevent users from seeing their own writes as missing (read-after-write inconsistency)?
   - **A:** Route reads to the primary for that user for a short window after writes (e.g., 10 seconds), or use sticky sessions so the same user always hits the same replica (which will eventually catch up). Critical: monitor replication lag and fail over to primary if lag is too high.

4) **Q:** Can you guarantee consistency with read replicas?
   - **A:** Only with synchronous replication, where the primary waits for the replica to confirm before committing—but this adds latency and reduces availability (if replica is down, writes block). Most systems use async replication and accept eventual consistency.

5) **Q:** What happens if a read replica fails?
   - **A:** Load balancer or connection pool detects failure and routes reads to healthy replicas or the primary. If you only had one replica, reads now overload the primary—so run at least 2 replicas for availability.

### Quick self-check (5 items)
- [ ] I can define read scaling and name 3 techniques (caching, replicas, denormalization).
- [ ] I can explain when to use replicas vs caching.
- [ ] I can describe replication lag and its impact.
- [ ] I can name 2 failure modes and their mitigations.
- [ ] I can explain read-after-write inconsistency and how to handle it.

## Links (NO duplication)
### Prerequisites
- [Distributed Systems Basics](distributed-systems-basics.md) — Understanding consistency models
- [Caching Fundamentals](caching-fundamentals.md) — Foundation for caching strategies
- (TODO) [Replication Basics](../databases/replication-basics.md) — Leader-follower patterns

### Related topics
- [PostgreSQL Scaling Playbook](../databases/postgresql-scaling-playbook.md) — Specific PostgreSQL read scaling
- [Redis Replication & Sentinel](../databases/redis-replication-sentinel.md) — Redis read scaling
- [Consistency Models](../databases/consistency-models.md) — Eventual vs strong consistency
- [PACELC](pacelc.md) — Latency vs consistency trade-offs during normal operation
- [Partitioning & Sharding](partitioning-and-sharding.md) — Write scaling complement

### Compare with
- [CQRS](../architecture/cqrs.md) — Separate read/write models (more architectural)
- [Denormalization](../databases/denormalization.md) — Optimizing reads via schema design
- [Multi-Region Replication](archetype-multi-region.md) — Geographic read scaling

## Notes / Inbox (optional)
- Common read:write ratios by industry: social media 100:1, ecommerce 20:1, financial 5:1
- Replication lag typically 100ms-5s; can spike to minutes during outages or bulk loads
- Caching typically handles 80-95% of read traffic if done well
- Read replicas typically 2-5; more than 5 indicates need for sharding or better caching
