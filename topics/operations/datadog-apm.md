---
title: Datadog APM
---

# Datadog APM

## Definition
Datadog APM (Application Performance Monitoring) is a cloud-based tool for monitoring, tracing, and visualizing application performance, distributed traces, and service dependencies.

## Why it matters
It helps teams detect bottlenecks, troubleshoot issues, and optimize performance in distributed systems.

## When to use / when not to use
Use for observability in microservices, cloud-native, or distributed architectures. Not needed for simple, monolithic apps.

## Trade-offs
- + Powerful distributed tracing and dashboards
- + Integrates with many platforms
- - Can be expensive at scale
- - High cardinality metrics may impact cost and performance

## Prerequisites
- [Observability basics](../operations/observability-basics.md)

## Related topics
- [SLI/SLO](../operations/service-level-indicator.md)
- [RED/USE dashboards](red-use-dashboards.md)

## Trap questions
1. What is a common pitfall when using Datadog APM?
   - High cardinality metrics can increase cost and reduce performance.
2. What does APM stand for?
   - Application Performance Monitoring.
3. When should you avoid Datadog APM?
   - For small projects where cost and complexity are not justified.
