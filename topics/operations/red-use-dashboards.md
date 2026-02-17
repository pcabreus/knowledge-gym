---
title: RED/USE Dashboards
---

# RED/USE Dashboards

## Definition
RED and USE are monitoring methodologies for building dashboards that focus on key metrics: Requests, Errors, Duration (RED) and Utilization, Saturation, Errors (USE).

## Why it matters
They provide a standardized way to monitor system health and performance, helping teams quickly identify bottlenecks and issues.

## When to use / when not to use
Use for SRE, DevOps, and operations teams to monitor services and infrastructure. Not needed for very simple systems.

## Trade-offs
- + Standardized, actionable metrics
- + Helps prioritize alerts and troubleshooting
- - May not capture all business-specific signals
- - Can be overwhelming if not tailored to context

## Prerequisites
- [Observability basics](../operations/observability-basics.md)

## Related topics
- [Datadog APM](datadog-apm.md)
- [SLI/SLO](../operations/service-level-indicator.md)

## Trap questions
1. What does RED stand for?
   - Requests, Errors, Duration.
2. What does USE stand for?
   - Utilization, Saturation, Errors.
3. What is a common pitfall with RED/USE dashboards?
   - Focusing only on technical metrics and missing business impact.
