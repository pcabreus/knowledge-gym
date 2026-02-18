---
id: fault-tolerance
title: "Fault Tolerance"
type: concept
status: learning
importance: 70
difficulty: medium
tags: [operations, reliability, resilience]
canonical: true
aliases: []
created_at: 2026-02-18
updated_at: 2026-02-18
---

# Fault Tolerance

## TL;DR (BLUF)
- Fault tolerance is the property of a system to continue operating correctly in the presence of hardware, software, or network faults.
- Achieve it through redundancy, isolation (bulkheads), graceful degradation, and automated recovery; trade-offs include cost and increased complexity.

## Definition
**What it is:** The ability of a system to tolerate faults without catastrophic failure, often by detecting faults, routing around them, or degrading functionality gracefully.  
**Key terms:** redundancy, graceful degradation, failover, bulkheads, retries, self-healing.

## Why it matters
- Fault-tolerant systems reduce user-visible outages and business impact when components fail.  
- Poor fault-tolerance leads to cascading failures, prolonged incidents, and reputational damage.

## Scope & Non-goals
**In scope:** design principles and operational controls that keep services usable during component failures.  
**Out of scope / NOT solved by this:** perfect availability or zero-error guarantees; fault tolerance accepts limited degraded behavior.

## Mental model / Intuition
- Treat faults as inevitable; design independent failure domains and fallbacks so that a single fault does not collapse the whole system.

## Decision rules (When to use / When not to use)
### Use it when
- You operate customer-facing or critical systems where outages have high cost.  
### Avoid it when
- You are building experimental, low-cost, or non-critical throwaway prototypes where added complexity is not justified.

## How I would use it (practical)
- **Context:** public API service with SLO-driven uptime targets.  
- **Steps:** identify failure domains → add redundancy and independent resource pools (bulkheads) → add graceful degradation paths → automate failover and recovery → run chaos experiments.  
- **What success looks like:** SLOs met during single-component failures and clear, observable degraded modes.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** user experience continuity, lower incident blast radius.  
- **Cons / Risks:** higher cost, more complex testing and deployment, potential for hidden shared dependencies.
### Alternatives
- For low-cost systems, prefer simpler approaches (best-effort availability) and focus on fast recovery over full redundancy.

## Failure modes & Pitfalls
- Hidden shared resources (single DB, single network path) that make replicated components fail together.  
- Over-reliance on retries without backoff causing cascading overload.  
- Incomplete or incorrect failover logic that leads to data loss or split-brain.

## Observability (How to detect issues)
- **Metrics:** error rate, latency p95/p99, circuit-breaker/open counts, failover/fallback counts.  
- **Logs:** failover events, degraded-mode indicators.  
- **Traces:** increased span durations during fallback paths.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Map failure domains and dependencies
  - [ ] Add independent resource pools and bulkheads
  - [ ] Implement deterministic failover and idempotent recovery
  - [ ] Run chaos/chaos-monkey experiments

## Mini example (if applicable)
```text
# Example: read cache with origin DB fallback. On cache miss or cache cluster failure, degrade to read-only DB with rate-limited queries and a graceful UI indicator.
```

## Common anti-patterns
- **Anti-pattern:** Replicating replicas that share the same underlying resource (storage, connection pool).  
  - **Why it’s bad:** replicas fail together, providing no real tolerance.  
  - **Better approach:** isolate resources, partition loads, and test failover.

## Interview readiness
### “Explain it like I’m teaching”
- Fault tolerance means designing systems so they continue to work (possibly degraded) when parts fail; use redundancy, isolation, and graceful degradation.

### Trap questions (with answers)
1) **Q:** Is adding more replicas always increasing fault tolerance?  
   - **A:** No—if replicas share critical resources they will fail together; true tolerance requires independent failure domains.  
2) **Q:** How do you validate fault tolerance?  
   - **A:** Through fault-injection (chaos) tests, dependency mapping, and measuring behavior during controlled failures.  
3) **Q:** How does CQRS help with write/read contention and fault tolerance?  
   - **A:** By separating read and write models you can isolate read load from write-side contention; but stores must be independently provisioned to avoid shared bottlenecks.

## Links (NO duplication)
### Prerequisites
- [Reliability basics](operations/reliability-basics.md)  
- [Bulkheads](operations/bulkheads.md)  
- [Read/Write Contention](../databases/read-write-contention.md)

### Related topics
- [Graceful Degradation](operations/graceful-degradation.md)  
- [Circuit Breaker](operations/circuit-breaker.md)  
- [CQRS](../architecture/cqrs.md)

### Compare with
- [False Tolerance](../operations/false-tolerance.md) — anti-pattern where apparent redundancy fails due to shared dependencies (if you previously added that file; consider removing).
