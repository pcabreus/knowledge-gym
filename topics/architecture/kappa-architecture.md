---
id: kappa-architecture
title: "Kappa Architecture"
type: concept
status: learning
importance: 55
difficulty: medium
tags: [architecture, data, stream]
canonical: true
aliases: [kappa-arch]
created_at: 2026-02-18
updated_at: 2026-02-18
---

# Kappa Architecture

## TL;DR (BLUF)
- Streaming-first architecture that processes data as a continuous stream; the same codebase handles both historical replay and real-time processing.
- Use when streaming processing can cover accuracy needs and you want a simpler operational model than Lambda.

## Definition
**What it is:** An architecture that treats the stream as the primary source of truth and processes data via a single streaming pipeline; replays are handled by replaying the stream.
**Key terms:** stream processing, replay, stream-as-source, exactly-once/at-least-once semantics.

## Why it matters
- Simplifies pipelines by removing separate batch and speed layers; reduces duplication and operational overhead compared to Lambda.

## Scope & Non-goals
**In scope:** systems where streaming frameworks can provide required correctness and latency.
**Out of scope / NOT solved by this:** workloads that require complex, long-running batch computations not suitable for streaming.

## Mental model / Intuition
- One pipeline to rule them all: write processors that can process historical data by replaying the stream and process live data continuously.

## Decision rules (When to use / When not to use)
### Use it when
- Your processing logic fits streaming model and can be made idempotent/deterministic for replays.
- You prefer operational simplicity over having separate specialized layers.
### Avoid it when
- Batch recomputation is easier or required for expensive global joins that streaming frameworks struggle with.

## How I would use it (practical)
- **Context:** event-driven product that needs both historical recompute and real-time projections but wants a single processing stack.
- **Steps:** choose a stream platform (Kafka, Pulsar), implement idempotent processors, provide replay tooling and monitoring.
- **What success looks like:** simple deployment model and predictable convergence when replaying data.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** reduced duplication, simpler ops, single source-of-truth processing model.
- **Cons / Risks:** streaming frameworks complexity, possibly harder for very large batch jobs.
### Alternatives
- **Lambda architecture:** if batch computes are much simpler or more performant for some workloads.

## Failure modes & Pitfalls
- Non-deterministic processing logic causing different results on replay.
- Backpressure and reprocessing complexity if upstream topics grow unbounded.

## Observability (How to detect issues)
- **Metrics:** processing lag, replay throughput, consumer offsets, deduplication counts.
- **Alerts:** persistent lag, high reprocessing error rates.

## Interview readiness
- Kappa is a streaming-first architecture that uses a single pipeline for both live processing and historical replays, trading Lambda’s dual-pipeline complexity for operational simplicity when streaming fits the use case.

## Links (NO duplication)
### Prerequisites
- [Kafka](../architecture/kafka.md)

### Related topics
- [Lambda architecture](lambda-architecture.md)
- [Materialized Views / CQRS](../system-design/archetype-materialized-views.md)

### Compare with
- [Lambda architecture](lambda-architecture.md) — Kappa uses a single streaming pipeline instead of separate batch+speed layers.
