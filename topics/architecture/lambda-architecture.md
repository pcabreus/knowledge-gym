---
id: lambda-architecture
title: "Lambda Architecture"
type: concept
status: learning
importance: 60
difficulty: medium
tags: [architecture, data, batch, stream]
canonical: true
aliases: [lambda-arch]
created_at: 2026-02-18
updated_at: 2026-02-18
---

# Lambda Architecture

## TL;DR (BLUF)
- Hybrid data-processing architecture: separate batch (accurate, high-latency) and speed (low-latency, approximate) layers, with a serving layer that merges results.
- Use when you need both high-throughput accurate re-computation and low-latency views; trade-off is implementation complexity and operational cost.

## Definition
**What it is:** An architecture that maintains a batch layer (full re-computation), a speed/real-time layer (incremental updates), and a serving layer that combines both to answer queries.
**Key terms:** batch layer, speed layer, serving layer, materialized view, re-computation, eventual consistency.

## Why it matters
- Enables low-latency queries while preserving long-term accuracy from batch recomputation.
- Common in analytics systems where correctness (e.g., corrections) and freshness are both important.

## Scope & Non-goals
**In scope:** large-scale analytics pipelines that require periodic recompute plus streaming updates.
**Out of scope / NOT solved by this:** small data workloads where a single processing model suffices; replacing transactional OLTP databases.

## Mental model / Intuition
- Think of two lenses: a slow but precise camera (batch) and a fast but fuzzy camera (speed); the serving layer overlays them so users see fresh results with eventual convergence to the accurate result.

## Decision rules (When to use / When not to use)
### Use it when
- You must correct historical errors (replayable batch) and also satisfy low-latency query needs.
- Data volume or processing complexity requires periodic full recompute.
### Avoid it when
- Data volumes are small or streaming-only processing provides sufficient accuracy.
- You cannot bear the operational cost of running dual pipelines.

## How I would use it (practical)
- **Context:** analytics dashboard that must show near-real-time metrics but also correct past anomalies after reprocessing.
- **Steps:** implement durable event/log source → batch recompute jobs (periodic) → streaming speed layer for incremental updates → serving layer merges views with clear precedence rules.
- **What success looks like:** low read-latency with observable convergence after batch runs; measurable freshness and accuracy metrics.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** freshness + accuracy, separation of concerns for batch vs stream processing.
- **Cons / Risks:** increased operational complexity, duplicated logic, harder testing and deployment.
### Alternatives
- **Kappa architecture:** prefer if streaming-only processing suffices and simplifies operations.
- **Materialized views / CQRS:** a simpler read-projection approach for many CQRS use cases.

## Failure modes & Pitfalls
- Diverging results between batch and speed layers due to inconsistent logic.
- Long convergence time after reprocessing leading to confusing UX.
- Duplication of business logic across layers causing maintenance burden.

## Observability (How to detect issues)
- **Metrics:** serving lag, batch job duration and success rate, discrepancy metrics between batch and speed outputs.
- **Logs:** batch job runs, streaming job offsets and errors.
- **Alerts:** failed reprocessing, divergence beyond acceptable thresholds.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Single canonical event/log source (e.g., CDC or append-only log)
  - [ ] Deterministic batch jobs and idempotent streaming processors
  - [ ] Monitoring for divergence and lag

## Mini example (if applicable)
```text
# Describe: events -> speed processor updates incremental counters; batch job recomputes full counters nightly; serving layer prefers batch values where available, otherwise speed.
```

## Common anti-patterns
- **Anti-pattern:** Keeping different business logic in batch vs speed.
  - **Why it’s bad:** causes permanent divergence and surprises.
  - **Better approach:** share a single canonical implementation (libraries/tests) and validate outputs.

## Interview readiness
### “Explain it like I’m teaching”
- Lambda splits processing into batch (accurate) and speed (fast) layers, then merges them so reads are fresh and eventually correct.

### Trap questions (with answers)
1) **Q:** Why not always use Lambda for analytics?
   - **A:** It doubles operational surface and duplicates logic; Kappa can be simpler when streaming covers accuracy needs.
2) **Q:** How do you handle conflicts between batch and speed results?
   - **A:** Define deterministic precedence (e.g., prefer batch after successful run) and expose freshness indicators.
3) **Q:** Does Lambda require event sourcing?
   - **A:** No, but a replayable append-only source simplifies batch recompute.

## Links (NO duplication)
### Prerequisites
- [Materialized Views / CQRS](../system-design/archetype-materialized-views.md)
- (TODO) event log / CDC topic (if not present)

### Related topics
- [Kappa architecture](kappa-architecture.md)
- [CQRS](../architecture/cqrs.md)

### Compare with
- [Kappa architecture](kappa-architecture.md) — streaming-only alternative that avoids dual pipelines.
