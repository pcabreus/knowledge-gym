---
id: two-stage-retrieval
title: "Two-Stage Retrieval"
type: pattern
status: learning
importance: 70
difficulty: medium
tags: [retrieval,search,cache,architecture]
canonical: true
aliases: []
created_at: 2026-02-23
updated_at: 2026-02-23
---

# Two-Stage Retrieval

## TL;DR (BLUF)
- Two-stage retrieval means: use a cheap, coarse-grained first-stage to narrow candidates, then apply an expensive, precise second-stage on that smaller set.
- Use when full-precision searches are expensive or large (e.g., vector search, fuzzy text, date ranges). Trade-off: extra complexity for big performance gains.

## Definition
**What it is:** A retrieval pattern where queries are executed in two phases: a fast, approximate or coarse filter (stage 1) that returns a candidate set, followed by an expensive, exact, or higher-fidelity rerank/filter (stage 2) applied only to those candidates.
**Key terms:** candidate generation, re-ranking, coarse filter, exact filter, index, locality-sensitive hashing (LSH), inverted index, bloom filter.

## Why it matters
- Reduces CPU/IO work by avoiding expensive operations across the full dataset.
- Enables low-latency results for complex queries (semantic search, fuzzy match, large graphs) while keeping high precision by rechecking candidates.

## Scope & Non-goals
**In scope:** search and retrieval problems across text, vectors, time-series, caches, and graphs where candidate sets can be pruned cheaply.
**Out of scope:** single-stage exact retrieval where data size is small enough to scan cheaply.

## Mental model / Intuition
- Think of it as “coarse sieve + fine comb”: the sieve throws away most irrelevant items cheaply; the comb inspects the leftovers carefully.

## Decision rules (When to use / When not to use)
### Use it when
- The exact operation is expensive (heavy scoring, vector distances, graph expansion).
- The candidate set can be efficiently approximated (e.g., coarse time buckets, prefix indexes, vector ANN, partitions).
### Avoid it when
- Dataset is small enough for exact queries to be cheap.
- The coarse filter frequently omits true positives (high recall loss).

## How I would use it (practical)
- **Context:** large event store where users query precise timestamp ranges and content similarity.
- **Steps:** 1) Build a coarse index (day buckets, prefix keys, ANN index) 2) Query coarse index to get candidates 3) Refilter/rerank candidates in memory using exact criteria 4) Return sorted results
- **What success looks like:** 90% reduction in disk reads and sub-100ms p95 latency for typical queries.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** Lower latency, reduced I/O/compute, scalable to large corpora.
- **Cons:** More complexity, potential recall loss if coarse stage is too aggressive, harder debugging/observability.
### Alternatives
- Single-stage exact index (simpler, but higher cost).  Use when data small.
- Materialized views for common queries (fast but high maintenance).

## Failure modes & Pitfalls
- Coarse index misses true positives (bad recall) — tune to favor recall.
- Candidate set too large (coarse stage ineffective) — redesign coarse index.
- Stale indexes between stages (consistency issues) — ensure synchronized updates.

## Observability (How to detect issues)
- **Metrics:** candidate reduction ratio (coarse_count / full_count), recall @k, p95 latency, error rate.
- **Logs:** coarse-query parameters, candidate counts, re-rank time per query.
- **Traces:** span for coarse stage + span for second-stage processing to find hotspots.

## Implementation notes (if applicable)
- Checklist:
  - [ ] Ensure coarse index favors recall over precision.
  - [ ] Measure candidate reduction and end-to-end latency.
  - [ ] Add alerts if candidate counts exceed thresholds.

## Mini example (pseudocode)
```pseudo
candidates = coarse_index.query(q)
results = []
for c in candidates:
  score = exact_score(c, q)
  if score >= threshold: results.append((c, score))
return sort_by_score(results)
```

## Common anti-patterns
- **Anti-pattern:** Aggressive coarse filters that optimize latency but drop many true positives.
  - **Why it’s bad:** lowers recall and user satisfaction.
  - **Better approach:** tune coarse filter or increase candidate window.

## Interview readiness
### “Explain it like I’m teaching”
- Two-stage retrieval means you first find a small set of possible matches quickly, then do the expensive exact check only on those — like checking only a shortlist of candidates instead of everyone.

### Trap questions (with answers)
1) **Q:** How do you pick between recall and precision for the coarse stage?
   - **A:** Prefer recall for the coarse stage; tune thresholds and measure recall@k. Adjust candidate window or expand buckets if recall drops.
2) **Q:** How do you keep stages consistent with updates?
   - **A:** Use synchronous writes to both indexes or design eventual-consistency checks; include sequence numbers/timestamps to detect staleness.
3) **Q:** When does two-stage retrieval add unnecessary complexity?
   - **A:** When data is small or exact queries are already fast — then single-stage exact search may be simpler and cheaper overall.

## Quick self-check (5 items)
- [ ] I can define it precisely in 2–3 sentences.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Search & Indexing Pipeline](system-design/archetype-search-indexing.md)
- [Caching fundamentals](system-design/caching-fundamentals.md)

### Related topics
- [Materialized Views / CQRS](system-design/archetype-materialized-views.md)
- [Vector search / ANN (TODO)](topics/_aliases/ann.md)
- [Inverted index (TODO)](topics/_aliases/inverted-index.md)

### Compare with
- [Kappa architecture](architecture/kappa-architecture.md) — two-stage is an indexing/query pattern; Kappa is a data-processing architecture.

## Notes / Inbox (optional)
- Examples and problem-solving files saved in `assessments/problem-solving/`.
