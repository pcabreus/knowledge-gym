---
id: brin-index
title: "BRIN Index (PostgreSQL)"
type: concept
status: learning
importance: 50
difficulty: medium
tags: [database, postgresql, indexing]
canonical: true
aliases: ["brin"]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# BRIN Index (PostgreSQL)

## TL;DR (BLUF)
- BRIN (Block Range Index) stores min/max summaries per block range instead of indexing every row.
- Use it for large tables with natural physical correlation (timestamps, auto-increment IDs).
- Trade-off: tiny index size and fast writes, but slower and less precise than B-Tree for random queries.

## Definition
**What it is:** A Block Range Index that stores summary metadata (min/max values) for contiguous block ranges, enabling fast pruning of non-matching blocks without indexing every row.  
**Key terms:** block range, physical correlation, min/max summary, index size, append-mostly workloads.

## Why it matters
- BRIN dramatically reduces index size (often 100-1000x smaller than B-Tree) for large tables.
- It's ideal for time-series or append-mostly data where physical order correlates with query filters.
- Wrong use on randomly ordered data results in full table scans.

## Scope & Non-goals
**In scope:**
- Large tables with natural physical correlation (created_at, auto-incrementing IDs).
- Append-mostly workloads with range queries on correlated columns.
- Reducing storage and write overhead for indexing.

**Out of scope / NOT solved by this:**
- Fast lookups on randomly ordered data (use B-Tree or GIN).
- High-selectivity equality queries (BRIN scans entire block ranges).
- Containment or complex operators (use GIN or GiST).

## Mental model / Intuition
- Imagine a library where books are physically sorted by publication year. Instead of cataloging every single book, you just label each shelf: "1990-1995", "1996-2000", etc. If you want books from 1998, you skip shelves labeled before 1996.
- BRIN works the same: it stores "this range of blocks contains values between X and Y". If your query needs values outside that range, the entire block range is skipped.

## Decision rules (When to use / When not to use)
### Use it when
- Table is large (millions+ rows) and you query by a column with natural physical correlation (e.g., `created_at`).
- Workload is append-mostly or data is partitioned by the indexed column.
- You need to reduce index size and write overhead.
- Query patterns use range filters (BETWEEN, >, <) on correlated columns.

### Avoid it when
- Data is randomly ordered or frequently updated in-place (physical correlation breaks).
- You need high selectivity for equality queries (B-Tree is faster).
- Column values are evenly distributed across blocks (BRIN won't prune effectively).
- Table is small enough that B-Tree overhead is acceptable.

## How I would use it (practical)
- **Context:** An events table with 100M rows, queried by `created_at` ranges, appended chronologically.
- **Steps:**
  1. Verify physical correlation: `SELECT ctid, created_at FROM events ORDER BY ctid LIMIT 1000;`
  2. Create BRIN index: `CREATE INDEX idx_events_created_brin ON events USING BRIN (created_at);`
  3. Validate with EXPLAIN: ensure bitmap scans and block range filtering.
  4. Monitor query performance vs B-Tree baseline.
- **What success looks like:** 95%+ reduction in index size, faster writes, acceptable query latency on range filters.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Extremely small index size (often < 1% of B-Tree).
  - Minimal write overhead (only summary updates on block fill).
  - Fast index creation and maintenance.
- **Cons / Risks:**
  - Slower and less precise than B-Tree for queries (scans entire block ranges).
  - Ineffective if physical correlation is low.
  - Not suitable for high-selectivity equality queries.
  - UPDATEs that move rows can degrade correlation over time.

### Alternatives
- **B-Tree index:** when data is randomly ordered or you need fast equality lookups.
- **Partitioning:** combine with BRIN per partition for better pruning.
- **No index:** if queries are rare or full scans are acceptable.
- **How to choose:** use BRIN for large, append-mostly, correlated data; B-Tree for random or high-selectivity access.

## Failure modes & Pitfalls
- **Physical correlation degrades:** frequent UPDATEs or random inserts break min/max boundaries, reducing pruning efficiency.
- **BRIN not used:** query planner ignores index if estimated cost is higher than seq scan.
- **Wrong column choice:** indexing a randomly distributed column results in no block pruning.
- **Insufficient autovacuum:** new data not reflected in BRIN summaries until summarization runs.

## Observability (How to detect issues)
- **Metrics:** 
  - Index size vs B-Tree comparison.
  - Query latency on range filters.
  - Bitmap heap scan counts and blocks pruned (from EXPLAIN).
- **Logs:** slow query logs for range queries on BRIN-indexed columns.
- **Traces:** DB query spans showing full scans instead of index usage.
- **Alerts:** sustained latency increase on time-range queries after data correlation degrades.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Verify physical correlation with `ctid` inspection
  - [ ] Create BRIN index on correlated column
  - [ ] Validate with EXPLAIN ANALYZE
  - [ ] Monitor index size and query performance
  - [ ] Schedule periodic VACUUM or autovacuum to update summaries
- **Performance notes**
  - Set `pages_per_range` based on table size and correlation (default 128).
  - Combine with table partitioning for better pruning.
  - Avoid on columns with frequent random UPDATEs.
- **Operational notes**
  - BRIN requires autovacuum or manual VACUUM to summarize new blocks.
  - Use `brin_summarize_new_values(index_oid)` to force summary updates.

## Mini example (if applicable)
```sql
-- Large events table with chronological inserts
CREATE TABLE events (
  id BIGSERIAL,
  created_at TIMESTAMPTZ NOT NULL,
  event_data JSONB
);

-- BRIN index on created_at (assuming physical correlation)
CREATE INDEX idx_events_created_brin ON events USING BRIN (created_at);

-- Query: events from last 7 days
EXPLAIN ANALYZE
SELECT * FROM events
WHERE created_at >= NOW() - INTERVAL '7 days';

-- Expected plan: Bitmap Heap Scan + Bitmap Index Scan (BRIN prunes old blocks)
```

## Common anti-patterns
- **Anti-pattern:** Using BRIN on randomly ordered data (e.g., user_id in multi-tenant table).
  - **Why it's bad:** no physical correlation → all blocks scanned → no benefit.
  - **Better approach:** use [B-Tree index](btree-index.md) or partition by user_id.
  
- **Anti-pattern:** Creating BRIN "just in case" without verifying correlation.
  - **Why it's bad:** wastes resources and confuses query planner.
  - **Better approach:** analyze `ctid` vs column values first; only add BRIN if correlation is strong.

- **Anti-pattern:** Expecting BRIN to replace B-Tree for all queries.
  - **Why it's bad:** BRIN is optimized for range scans on correlated data, not general-purpose indexing.
  - **Better approach:** use BRIN for correlated range queries; keep B-Tree for equality/random access.

## Interview readiness
### "Explain it like I'm teaching"
BRIN is a space-efficient PostgreSQL index that stores min/max summaries for block ranges instead of indexing every row. It's ideal for large, append-mostly tables with natural physical correlation—like timestamps or auto-incrementing IDs. The trade-off is smaller index size and faster writes, but slower and less precise queries compared to B-Tree.

### Trap questions (with answers)
1) **Q:** Is BRIN always better than B-Tree because it's smaller?
   - **A:** No. BRIN is only effective when data has strong physical correlation. For randomly ordered data, it provides no pruning benefit and B-Tree will be much faster.

2) **Q:** Does BRIN automatically update when new rows are inserted?
   - **A:** Not immediately. BRIN summaries are updated during autovacuum or manual VACUUM. You can force updates with `brin_summarize_new_values()`.

3) **Q:** Can BRIN speed up equality queries like `WHERE id = 12345`?
   - **A:** Not reliably. BRIN scans entire block ranges that might contain the value. B-Tree is much better for high-selectivity equality queries.

4) **Q:** What happens to BRIN effectiveness if I UPDATE rows frequently?
   - **A:** Physical correlation can degrade, especially if UPDATEs cause row movement. This reduces block pruning efficiency, making BRIN less useful over time.

5) **Q:** Should I use BRIN on small tables?
   - **A:** No. BRIN's benefits (small index size, low write overhead) only matter for large tables. On small tables, B-Tree overhead is negligible and much faster.

### Quick self-check (5 items)
- [ ] I can define BRIN precisely in 2–3 sentences.
- [ ] I can state when to use it and when not to (correlation + size).
- [ ] I can explain at least 2 trade-offs (size vs precision, writes vs reads).
- [ ] I can give a concrete example from memory (timestamp column on large table).
- [ ] I can name 1 failure mode and how to detect it (correlation degrades → slow queries).

## Links (NO duplication)
### Prerequisites
- [Index](index.md)
- [B-Tree index](btree-index.md)

### Related topics
- [PostgreSQL MVCC](postgresql-mvcc.md)
- [PostgreSQL vacuum & autovacuum](postgresql-vacuum-autovacuum.md)
- [EXPLAIN](explain.md)

### Compare with
- [B-Tree index](btree-index.md) — BRIN trades query speed for index size on correlated data; B-Tree is faster but larger.
- [GIN index](gin-index.md) — GIN for containment queries; BRIN for range queries on correlated columns.

## Notes / Inbox (optional)
- Consider adding example showing `ctid` correlation analysis.
- Mention `pages_per_range` tuning for different table sizes.
