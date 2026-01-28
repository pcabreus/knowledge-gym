---
id: archetype-security-access-control
title: "Security & Access Control at Scale"
type: pattern
status: learning
importance: 95
difficulty: hard
tags: [system-design, security, access-control, rbac, abac, authorization, authentication]
canonical: true
aliases: [authentication-authorization, rbac-abac, policy-engines]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Security & Access Control at Scale

## TL;DR
- Control who can do what, with auditability and low latency at scale.
- Key challenges: complex rules (roles, resources, contexts), performance of authorization checks, audit requirements.
- Solutions: policy-as-data (central policy service), RBAC for simple / ABAC for complex, audit log as append-only store.

## Where it hurts (why it hurts)
1. **Complex rules (roles, resources, contexts)**: "Admin can view payroll but not edit contracts; manager can view only their team"
   - **Solution**: RBAC (simple roles) or ABAC (attribute-based for complex contexts); policy engine (OPA, Zanzibar)
2. **Performance of authorization checks**: Every request checks permissions → latency overhead
   - **Solution**: Cache decisions (with TTL), precompute common checks, policy decisions as data
3. **Audit requirements**: Compliance requires "who accessed what when" logs
   - **Solution**: Append-only audit log (immutable), centralized logging service

## Decision rules
- **Use when**: Enterprise SaaS, finance, healthcare (compliance), multi-tenancy with permissions
- **Avoid when**: Single admin user (no roles), public read-only API

## Trade-offs
- **RBAC (Role-Based)**: Simple, easy to reason; limited (role explosion for complex rules)
- **ABAC (Attribute-Based)**: Flexible, context-aware (time, location, resource attributes); complex to debug
- **Choose**: Simple roles → RBAC; complex context → ABAC

## Explicit example
Enterprise SaaS: Users have roles (Admin, Manager, Employee). Admin can view all data; Manager can view only their team's data; Employee can view only their own. ABAC rule: `allow if (role == Manager AND resource.team_id == user.team_id) OR (role == Admin)`. Policy evaluated on each request; cached for 60s. Audit log: every access logged with (user, resource, action, outcome, timestamp).

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [Multi-Tenancy](archetype-multi-tenancy.md), [Rate Limiting](archetype-rate-limiting-quotas.md)  
**Technologies**: OPA (Open Policy Agent), Google Zanzibar, AWS IAM, RBAC, ABAC
