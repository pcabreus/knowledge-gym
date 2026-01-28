---
id: archetype-data-retention
title: "Data Retention, Archival & Cost-Control"
type: pattern
status: learning
importance: 80
difficulty: medium
tags: [system-design, data-retention, archival, lifecycle, cold-storage, tiered-storage]
canonical: true
aliases: [tiered-storage, ilm, lifecycle-policies, cold-storage]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Data Retention, Archival & Cost-Control

## TL;DR
- Manage data growth with retention rules and storage tiers to control costs.
- Key challenges: cost explosion, compliance requirements (GDPR delete vs immutable logs), querying mixed hot/cold data.
- Solutions: rollups/downsampling, lifecycle policies (auto move/delete), separate audit store with restricted access.

## Where it hurts (why it hurts)
1. **Cost explosion**: Keeping raw clickstream forever → petabytes → unaffordable
   - **Solution**: Retention policies (raw 7d, rollups 1y, cold archive for compliance)
2. **Compliance requirements**: GDPR requires delete; audit logs must be immutable
   - **Solution**: Separate hot store (deletable) + cold audit store (immutable, restricted)
3. **Querying mixed hot/cold data**: Users want "last 2 years" → slow query across tiers
   - **Solution**: Federated query (query hot + cold); or redirect old queries to archive UI

## Decision rules
- **Use when**: High-volume data (logs, metrics, events), retention requirements differ by age, cost optimization critical
- **Avoid when**: Low volume (< 1TB total), uniform retention (delete all at once)

## Trade-offs
- **Hot storage**: Fast, expensive → recent data
- **Cold storage**: Slow (minutes to retrieve), cheap → old data / compliance
- **Rollups**: Pre-aggregated data → cheap, fast queries but less detail

## Explicit example
Clickstream: Raw events massive. Retain raw 7 days (hot, queryable), hourly rollups 1 year (warm), raw archive to S3 Glacier for 7 years (cold, compliance). Queries for recent data hit hot store (fast); historical analysis uses rollups (fast, less detail); compliance audit uses Glacier (slow, rare).

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [Time-Series Aggregation](archetype-time-series-aggregation.md), [Materialized Views](archetype-materialized-views.md)
