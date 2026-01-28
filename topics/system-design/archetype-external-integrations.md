---
id: archetype-external-integrations
title: "External Integrations (Unreliable Dependencies)"
type: pattern
status: learning
importance: 95
difficulty: medium
tags: [system-design, third-party, webhooks, circuit-breaker, resilience, integrations]
canonical: true
aliases: [webhook-processing, third-party-api-integration, partner-systems]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# External Integrations (Unreliable Dependencies)

## TL;DR
- Interact with external systems (third-party APIs, webhooks, partners) that have unknown latency and failure modes.
- Key challenges: timeouts and partial failures, retries causing duplicates, provider rate limits.
- Solutions: timeouts + circuit breaker + retries with jitter, async processing, DLQ + replay.

## Where it hurts (why it hurts)
1. **Timeouts and partial failures**: Tax API call during checkout is slow/down → lose conversions or risk compliance
   - **Solution**: Timeouts (aggressive), circuit breaker (fail fast), async workflows (decouple user path)
2. **Retries causing duplicates**: Retry webhook → partner receives duplicate → double processing
   - **Solution**: Idempotency on both sides (send idempotency key); dedupe at recipient
3. **Provider rate limits**: Hit API rate limit → all requests fail → service degraded
   - **Solution**: Local rate limiting (stay under provider limits), backoff, cache responses

## Decision rules
- **Use when**: Payment providers, KYC/compliance APIs, shipping APIs, CRM integrations, webhooks
- **Avoid when**: Fully controlled internal services (different patterns apply)

## Trade-offs
- **Sync integration (block user)**: Simple but user waits for slow/failed external call
- **Async (decouple)**: User doesn't wait but eventual outcome (webhook/polling)
- **Choose**: Critical path (checkout) → async if possible; non-critical → sync acceptable

## Explicit example
Checkout calls tax API for VAT calculation. API is slow (2s p99) or down. If checkout blocks → lost conversions. Solution: Call tax API async (after order placed), email user final price; or use cached/estimated tax (notify if adjustment needed). Circuit breaker: after 5 failures, stop calling for 30s; return estimated tax.

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [Idempotency](archetype-idempotency-dedup.md), [Workflow Orchestration](archetype-workflow-orchestration.md)  
**Patterns**: Circuit Breaker, Bulkheads, Timeouts, DLQ (see [Reliability Basics](../operations/reliability-basics.md))
