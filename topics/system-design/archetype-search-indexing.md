---
id: archetype-search-indexing
title: "Search & Indexing Pipeline"
type: pattern
status: learning
importance: 85
difficulty: medium
tags: [system-design, search, indexing, inverted-index, elasticsearch]
canonical: true
aliases: [inverted-index, indexing-service, retrieval-ranking]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Search & Indexing Pipeline

## TL;DR
- Maintain a search index separate from transactional storage for fast full-text/multi-field queries.
- Key challenges: freshness (index lag), ranking complexity, reindexing costs.
- Solutions: event-driven indexing + CQRS, rebuild from source, separate ranking service.

## Where it hurts (why it hurts)
1. **Freshness (index lag)**: Inventory changes; search results show out-of-stock items → bad UX
   - **Solution**: Event-driven indexing (near real-time); monitor lag
2. **Ranking complexity**: Simple keyword match → poor relevance; users can't find what they need
   - **Solution**: Separate ranking service; A/B test ranking models
3. **Reindexing costs**: Schema change → rebuild entire index → hours of downtime
   - **Solution**: Blue-green indexing (build new index, cutover); incremental updates

## Decision rules
- **Use when**: Full-text search, multi-field filtering/sorting, product catalogs, document search
- **Avoid when**: Simple exact-match queries (DB index sufficient), < 1000 records

## Trade-offs
- **Real-time indexing**: Fresh but expensive (write amplification)
- **Batch indexing**: Cheaper but stale
- **Choose**: User-facing search → real-time; analytics → batch

## Explicit example
E-commerce: Users search "red shoes size 10." DB query slow (full-table scan). Elasticsearch index: products indexed by (title, description, category, attributes). Events (ProductCreated, ProductUpdated) trigger index updates. Query hits index (ms), not DB (seconds).

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [Materialized Views](archetype-materialized-views.md), [CQRS](../architecture/cqrs.md), [Event-Driven](../architecture/event-driven-basics.md)
