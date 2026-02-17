---
title: Webhooks
---

# Webhooks

## Definition
Webhooks are HTTP callbacks that allow one system to notify another system in real time when an event occurs, by sending an HTTP POST request to a configured URL.

## Why it matters
They enable real-time integrations and automation between systems without polling.

## When to use / when not to use
Use for event-driven integrations, notifications, or third-party API callbacks. Not ideal for critical, high-reliability delivery without retries or verification.

## Trade-offs
- + Real-time, low-latency communication
- + Simple to implement
- - Delivery not guaranteed (unless retries/verification are added)
- - Security risks if not validated

## Prerequisites
- [HTTP](../operations/http.md)

## Related topics
- [API Gateway](../system-design/api-gateway.md)
- [Third-party integration resilience](../operations/third-party-integration-resilience.md)

## Trap questions
1. What is a webhook?
   - An HTTP callback triggered by an event in a source system.
2. What is a common security risk with webhooks?
   - Not verifying the signature or source of the request.
3. How can you make webhooks more reliable?
   - Implement retries and idempotency.
