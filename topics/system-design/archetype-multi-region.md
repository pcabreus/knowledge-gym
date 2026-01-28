---
id: archetype-multi-region
title: "Multi-Region, Replication & Consistency"
type: pattern
status: learning
importance: 90
difficulty: hard
tags: [system-design, multi-region, geo-replication, active-active, active-passive, dr]
canonical: true
aliases: [geo-replication, active-active, active-passive, disaster-recovery]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Multi-Region, Replication & Consistency

## TL;DR
- Serve users across regions and survive regional failures with geo-distributed infrastructure.
- Key challenges: data consistency across regions, failover complexity, latency trade-offs.
- Solutions: active-passive with failover, CRDT/conflict resolution for multi-writer, region-local reads + async replication.

## Where it hurts (why it hurts)
1. **Data consistency across regions**: Profile updated in EU and US simultaneously → conflicts
   - **Solution**: Active-passive (one writer) or CRDTs / conflict resolution (multi-writer)
2. **Failover complexity**: Region failure → manual intervention → RTO in hours
   - **Solution**: Automated health checks + failover; DR drills
3. **Latency trade-offs**: Writing to remote region → high latency for some users
   - **Solution**: Region-local reads + async replication for writes; accept eventual consistency

## Decision rules
- **Use when**: Global user base, disaster recovery (DR) requirements, regulatory data residency
- **Avoid when**: Regional user base, cost/complexity outweighs DR needs

## Trade-offs
- **Active-passive**: Simple, strong consistency; higher latency for remote users
- **Active-active (multi-writer)**: Low latency everywhere; conflict resolution complexity
- **Choose**: Simplicity / strong consistency → active-passive; global low latency → active-active (if conflicts manageable)

## Explicit example
Global app: EU users + US users. Active-passive: US is primary (writes); EU is replica (read-only). If US fails, manual failover to EU (RTO 15 min). Active-active: both regions accept writes; use CRDT (e.g., last-write-wins with timestamps) to resolve conflicts.

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [CAP Theorem](cap-theorem.md), [PACELC](pacelc.md), [Consistency Models](../databases/consistency-models.md)
