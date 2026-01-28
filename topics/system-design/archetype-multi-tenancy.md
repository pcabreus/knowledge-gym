---
id: archetype-multi-tenancy
title: "Multi-Tenancy & Isolation"
type: pattern
status: learning
importance: 90
difficulty: hard
tags: [system-design, multi-tenancy, saas, noisy-neighbor, isolation]
canonical: true
aliases: [tenant-isolation, noisy-neighbor, per-tenant-quotas]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Multi-Tenancy & Isolation

## TL;DR
- Serve many customers on shared infrastructure safely without one impacting others.
- Key challenges: noisy neighbor impacts others, data isolation & compliance, per-tenant scaling/billing.
- Solutions: per-tenant quotas / rate limiting, partitioning by tenant ID, workload isolation (queues, pools).

## Where it hurts (why it hurts)
1. **Noisy neighbor impacts others**: One tenant runs huge payroll export → slows API for all
   - **Solution**: Per-tenant rate limiting / quotas; separate worker pools for heavy operations
2. **Data isolation & compliance**: GDPR/compliance requires strict tenant separation
   - **Solution**: Tenant ID in every key + authz enforcement; DB-per-tenant for strictest isolation
3. **Per-tenant scaling and billing**: Need to track usage and costs per tenant
   - **Solution**: Instrumentation per tenant ID; quotas + usage metering

## Decision rules
- **Use when**: SaaS platforms, B2B products, shared infrastructure serving multiple customers
- **Avoid when**: Single customer (no multi-tenancy), B2C with uniform users (no "tenants")

## Trade-offs
- **Shared DB**: Cheaper, easier ops; risk of noisy neighbor
- **DB-per-tenant**: Strict isolation, independent scaling; operational burden
- **Choose**: Most start shared DB + quotas; move to DB-per-tenant for large/regulated tenants

## Explicit example
SaaS HR platform: Tenant A runs massive payroll export (10k rows) → locks DB tables → Tenant B's API calls timeout. Solution: Per-tenant rate limits + async job queue with quotas. Tenant A's job queued separately; doesn't block Tenant B.

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [Rate Limiting](archetype-rate-limiting-quotas.md), [Security & Access Control](archetype-security-access-control.md)
