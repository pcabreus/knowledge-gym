---
title: SQS FIFO
---

# SQS FIFO

## Definition
SQS FIFO (First-In-First-Out) is an AWS managed message queue that guarantees the order of message delivery and supports deduplication.

## Why it matters
It enables reliable, ordered processing of events in distributed systems, critical for workflows that require strict sequencing.

## When to use / when not to use
Use for workflows where order and exactly-once processing are required. Not needed for unordered, high-throughput workloads (use SQS Standard).

## Trade-offs
- + Guarantees order and deduplication
- + Managed by AWS, scales automatically
- - Lower throughput than SQS Standard
- - Deduplication window limits

## Prerequisites
- [Message delivery semantics](../architecture/message-delivery-semantics.md)

## Related topics
- [SNS](sns.md) (TODO if missing)
- [Retries and Backoff](../operations/retries-and-backoff.md)

## Trap questions
1. What is a key feature of SQS FIFO?
   - Guarantees message order and deduplication.
2. When should you avoid SQS FIFO?
   - For high-throughput, unordered workloads.
3. What is a common pitfall with SQS FIFO?
   - Exceeding the deduplication window or message group limits.
