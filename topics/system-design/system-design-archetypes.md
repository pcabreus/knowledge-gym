---
id: system-design-archetypes
title: "System Design Archetypes"
type: concept
status: learning
importance: 95
difficulty: hard
tags: [system-design, architecture, patterns, scaling]
canonical: true
aliases: []
created_at: 2026-01-28
updated_at: 2026-01-28
---

# System Design Archetypes

## TL;DR (BLUF)
- **System design archetypes** are recurring problem patterns with known solution spaces that appear across distributed systems.
- Mastering these helps you quickly identify the core challenges in any system and apply proven solutions.
- Each archetype has predictable "where it hurts" failure modes and pattern combinations that solve them.

## Definition
**What it is:** A **system design archetype** is a recurring problem pattern in distributed systems with well-understood failure modes, trade-offs, and solution patterns. Unlike individual patterns (e.g., Circuit Breaker), archetypes represent complete problem domains (e.g., "handling unreliable external dependencies").

**Key terms:**
- **Archetype**: The recurring problem pattern
- **Failure modes**: Where and how the system breaks under stress
- **Solution patterns**: Known patterns that address the archetype's challenges
- **Trade-offs**: The unavoidable tensions in solving the problem

## Why it matters
- **Interview readiness**: System design interviews test your ability to recognize archetypes and apply appropriate patterns.
- **Real-world impact**: Most production incidents map to a mishandled archetype (e.g., missing backpressure, inadequate idempotency, hot partition).
- **Decision velocity**: Recognizing archetypes helps you make faster, better-informed architectural decisions.
- **Learning acceleration**: Studying archetypes provides a mental framework to organize individual patterns and technologies.

## Scope & Non-goals
**In scope:**
- 18 core archetypes covering most distributed system challenges
- Failure modes, solution patterns, and real examples for each
- Cross-references to related archetypes and patterns

**Out of scope / NOT solved by this:**
- Implementation details of specific technologies
- Complete pattern catalog (see individual pattern topics for depth)
- Domain-specific archetypes (e.g., ML systems, blockchain)

## Mental model / Intuition
Think of archetypes as **disease categories** in medicine:
- The archetype is the disease class (e.g., "infection")
- Failure modes are symptoms (e.g., "fever, inflammation")
- Solution patterns are treatments (e.g., "antibiotics, rest")
- Trade-offs are side effects (e.g., "antibiotic resistance risk")

When designing systems, you're diagnosing which archetypes apply and prescribing appropriate patterns.

## The 18 Core Archetypes

### Data & Event Processing
1. [High-Throughput Event Ingestion](archetype-event-ingestion.md) — Handling continuous event streams reliably
2. [Time-Series Aggregation](archetype-time-series-aggregation.md) — Windowed computations over time-stamped data
3. [Materialized Views / CQRS](archetype-materialized-views.md) — Read-optimized projections from write sources
4. [Search & Indexing Pipeline](archetype-search-indexing.md) — Maintaining searchable indexes separate from source data

### Traffic & Resource Management
5. [Distributed Caching](archetype-distributed-caching.md) — Reducing latency and load with cache layers
6. [Rate Limiting & Quotas](archetype-rate-limiting-quotas.md) — Protecting systems from overload
7. [Hot Partition / Celebrity Problem](archetype-hot-partition.md) — Handling skewed traffic distribution
8. [Multi-Tenancy & Isolation](archetype-multi-tenancy.md) — Serving many customers safely on shared infrastructure

### Reliability & Correctness
9. [Idempotency & Deduplication](archetype-idempotency-dedup.md) — Retry-safe operations
10. [Concurrency Control](archetype-concurrency-control.md) — Preventing race conditions and conflicts
11. [Ordering Guarantees](archetype-ordering-guarantees.md) — Per-key sequencing in distributed systems
12. [External Integrations](archetype-external-integrations.md) — Handling unreliable third-party dependencies

### Communication & Delivery
13. [Fan-Out / Broadcast Delivery](archetype-fan-out.md) — Delivering one event to many targets
14. [Workflow Orchestration](archetype-workflow-orchestration.md) — Long-running multi-step processes
15. [Real-Time Communication](archetype-real-time-communication.md) — WebSockets, presence, and stateful connections

### Cross-Cutting Concerns
16. [Multi-Region Replication](archetype-multi-region.md) — Geo-distribution and disaster recovery
17. [Data Retention & Lifecycle](archetype-data-retention.md) — Managing growth with tiered storage
18. [Security & Access Control](archetype-security-access-control.md) — Authentication, authorization, and audit at scale

## How to use this knowledge

### During system design interviews
1. **Listen for archetype signals** in requirements (e.g., "spiky traffic" → Backpressure + Caching)
2. **Call out the archetype** explicitly: "This is a classic hot partition problem"
3. **Discuss failure modes** first, then patterns that address them
4. **Compare alternatives** with clear decision rules

### In real projects
1. **Map requirements to archetypes** during design phase
2. **Review each archetype's checklist** to avoid known pitfalls
3. **Use archetype cross-references** to find related concerns
4. **Monitor archetype-specific metrics** in production

## Decision rules (When to use / When not to use)
### Study archetypes when
- Preparing for system design interviews (especially senior+ roles)
- Designing distributed systems from scratch
- Debugging production incidents (map symptoms to archetypes)
- Leading architecture reviews

### Don't over-apply when
- Building simple monoliths (most archetypes assume distribution)
- Problem space is truly novel (defer to first principles)
- Scale doesn't justify complexity (start simple, add patterns as needed)

## Trade-offs & Alternatives
### Studying archetypes
- **Pros:**
  - Faster pattern recognition
  - Organized mental model
  - Better interview performance
  - Reduced production incidents
- **Cons / Risks:**
  - Can lead to over-engineering if applied prematurely
  - Requires significant upfront study time
  - May miss domain-specific nuances

### Alternatives
- **Pattern-by-pattern learning:** Deeper on individual patterns, slower to see connections
- **Technology-first learning:** Tied to specific tools, less transferable
- **Project-based learning:** Practical but incomplete coverage

**How to choose:** Use archetypes as a framework, learn individual patterns for depth, validate with real projects.

## Failure modes & Pitfalls
- **Premature optimization:** Applying complex patterns before scale demands them
- **Archetype tunnel vision:** Forcing a problem into a known archetype when it doesn't fit
- **Ignoring trade-offs:** Choosing a pattern without understanding its costs
- **Missing archetype combinations:** Real systems often combine 3-5 archetypes; don't solve in isolation

## Interview readiness
### "Explain it like I'm teaching"
System design archetypes are recurring problem patterns in distributed systems. Instead of memorizing every possible system, you learn the 15-20 core problem types—like "handling spiky traffic" or "ensuring exactly-once processing"—and the proven patterns that solve each. When you encounter a new system, you recognize which archetypes apply and compose the right solutions. It's like knowing disease categories in medicine: once you diagnose the problem type, you know which treatments to consider.

### Trap questions (with answers)
1) **Q:** Aren't archetypes just patterns?
   - **A:** No. Patterns are solutions (e.g., Circuit Breaker). Archetypes are problem domains (e.g., "unreliable external dependencies"). One archetype typically requires multiple patterns working together.

2) **Q:** Do I need to apply all relevant archetypes to every system?
   - **A:** No. Apply only what current scale and requirements demand. Over-engineering is a failure mode. Start simple, add patterns as load/complexity justifies them.

3) **Q:** How do archetypes relate to the actual technologies I'll use?
   - **A:** Archetypes are technology-agnostic problem spaces. For example, "Event Ingestion" applies whether you use Kafka, Kinesis, or RabbitMQ. The archetype teaches you what to look for; technology docs teach you how to implement.

4) **Q:** Can a system have multiple archetypes?
   - **A:** Absolutely. Real systems typically combine 3-7 archetypes. For example, a notification system involves: Event Ingestion, Fan-Out, Idempotency, Rate Limiting, and External Integrations. The skill is recognizing all applicable archetypes and ensuring their patterns compose cleanly.

5) **Q:** What if my problem doesn't fit any archetype?
   - **A:** Truly novel problems are rare. First, ensure you understand the archetypes deeply—most "novel" problems are actually combinations of known archetypes. If still novel, fall back to first principles and document your solution for others.

## Links
**Prerequisites:**
- [Distributed Systems Basics](distributed-systems-basics.md)
- [Performance Basics](performance-basics.md)
- [Caching Fundamentals](caching-fundamentals.md)

**Related topics:**
- [Microservices Design Basics](../architecture/microservices-design-basics.md)
- [Event-Driven Architecture](../architecture/event-driven-basics.md)
- [Reliability Basics](../operations/reliability-basics.md)
- [Observability](../operations/observability.md)

**Individual archetypes:** See list above with links to each of the 18 archetype files.
