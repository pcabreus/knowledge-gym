---
id: false-tolerance
title: "False Tolerance"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [operations, reliability, resilience]
canonical: true
aliases: []
created_at: 2026-02-18
updated_at: 2026-02-18
---

# False Tolerance

## TL;DR (BLUF)
- An anti-pattern where a system appears fault-tolerant under common conditions but fails catastrophically under correlated or uncommon failures because redundancy or mitigation is incomplete.
- Watch for hidden single points-of-failure, shared resource exhaustion, or masking that gives a false sense of safety.

## Definition
**What it is:** When redundancy or safeguards give the illusion of resilience but do not cover realistic failure modes (e.g., shared dependencies, resource competition), producing surprising outages.
**Key terms:** single point-of-failure, redundancy, correlated failure, resource contention.

## Why it matters
- Leads teams to under-invest in real resilience measures and to be surprised by incidents that bypass superficial safeguards.

## Scope & Non-goals
**In scope:** design reviews, incident postmortems, reliability planning.
**Out of scope / NOT solved by this:** normal transient failures that proper redundancy already covers.

## Mental model / Intuition
- Like a building with multiple exits that all rely on the same corridor: exits exist, but if the corridor is blocked none work.

## Decision rules (When to use / When not to use)
### Use it when
- Evaluating resilience: ask whether redundancy actually isolates failure domains and resource boundaries.
### Avoid it when
- You assume that simple replication equals resilience without checking shared dependencies and resource contention.

## How I would use it (practical)
- **Context:** design review for high-traffic service.
- **Steps:** enumerate dependencies → map failure domains → test correlated failures (chaos) → enforce isolation (bulkheads, separate databases, queues).
- **What success looks like:** each redundancy path survives realistic dependency failures and resource exhaustion tests.

## Trade-offs & Alternatives
### Trade-offs
- **Pros of addressing it:** fewer surprising outages, clearer operational responsibility.
- **Cons / Risks:** additional cost and architectural complexity to truly isolate domains.

## Failure modes & Pitfalls
- Hidden shared resources (single connection pool, single disk, single region) that cause simultaneous failure of redundant components.

## Observability (How to detect issues)
- **Metrics:** correlated error spikes across redundant instances, exhaustible resource metrics (connection pools, CPU, file descriptors).
- **Alerts:** multiple paths failing simultaneously, unexpected saturation of shared resources.

## Implementation notes (if applicable)
- Prefer explicit partitioning (bulkheads), independent resource pools, and chaos experiments to validate assumptions.

## Interview readiness
- Explain that false tolerance is an anti-pattern where apparent redundancy lacks independent failure domains — demonstrate with an example (replicas sharing a single storage mount or DB connection pool).

## Trap questions (with answers)
1) **Q:** Is adding replicas always increasing tolerance?
   - **A:** Not if they share the same resource or dependency—then they fail together.

2) **Q:** How do you prove tolerance is real?
   - **A:** Run chaos tests, validate independence of failure domains, measure resource saturation under load.

3) **Q:** How does CQRS help against false tolerance?
   - **A:** By separating read and write models you can isolate read workloads from write contention; however you must ensure read/write stores don’t share the same bottlenecks.

## Links (NO duplication)
### Prerequisites
- [Read/Write Contention](../databases/read-write-contention.md)
- [Bulkheads](../operations/bulkheads.md)

### Related topics
- [CQRS](../architecture/cqrs.md)
- [Graceful Degradation](../operations/graceful-degradation.md)
