---
id: strangler-fig
title: "Strangler Fig Pattern"
type: pattern
status: learning
importance: 75
difficulty: hard
tags: [architecture, migration, legacy, refactoring]
canonical: true
aliases: [strangler pattern, incremental migration]
created_at: 2026-01-26
updated_at: 2026-01-26
---

# Strangler Fig Pattern

## TL;DR (BLUF)
- Incrementally migrate a legacy system by routing traffic to new services while keeping old system running.
- Use it to avoid risky "big bang" rewrites of monoliths or legacy systems.
- Trade-off: long migration period with increased complexity managing both systems.

## Definition
**What it is:** A migration strategy where you gradually replace parts of a legacy system by intercepting requests (via proxy/gateway), routing some to new services and others to the old system, until the old system is fully "strangled" and can be decommissioned.

**Key terms:** incremental migration, legacy modernization, proxy layer, routing, parallel run, big bang rewrite (anti-pattern), brownfield development.

## Why it matters
- Reduces risk of failed rewrites (avoid "second system syndrome").
- Allows continuous delivery: migrate one feature at a time.
- Keeps business running during migration (no downtime).
- Provides rollback path (route back to legacy if new service fails).

## Scope & Non-goals
**In scope:** routing strategies, proxy/gateway setup, data migration, dual-write handling, cutover planning.

**Out of scope / NOT solved by this:**
- Data consistency during migration (see [Dual-write](dual-write-pattern.md))
- Domain modeling (see [Domain-Driven Design](domain-driven-design.md))
- Legacy system decommissioning timeline

## Mental model / Intuition
- Named after strangler fig tree: grows around host tree, eventually replacing it.
- In systems: new microservices "wrap around" monolith, gradually taking over functionality.
- Like renovating a house room by room while living in it.

## Decision rules (When to use / When not to use)
### Use it when
- Legacy system is large and risky to rewrite all at once.
- Business cannot afford downtime or big-bang cutover.
- You need to deliver value incrementally (new features in new services).
- Team lacks full understanding of legacy (discover as you migrate).
- Legacy has mixed quality (migrate valuable parts, discard cruft).

### Avoid it when
- Legacy is small enough to rewrite quickly (< 6 months).
- Legacy is stable and low-maintenance (no business case for migration).
- Team is too small to manage dual systems.
- Data migration complexity is prohibitive.

## How I would use it (practical)
- **Context:** Migrating monolithic e-commerce app to microservices.
- **Steps:**
  1) **Setup proxy/gateway:** Deploy [API Gateway](../system-design/api-gateway.md) in front of monolith.
     ```
     Client → API Gateway → (route decision) → New Services OR Legacy Monolith
     ```
  2) **Identify migration slice:** Start with low-risk, high-value feature (e.g., product catalog).
  3) **Build new service:** Create `product-service` (microservice).
  4) **Dual-write (if needed):** Write to both legacy DB and new service DB during transition.
     ```go
     // Temporarily write to both
     legacyDB.SaveProduct(product)
     newProductService.CreateProduct(product)
     ```
  5) **Route reads to new service:** Configure gateway:
     ```
     GET /products/* → product-service (new)
     POST /orders/* → monolith (legacy)
     ```
  6) **Monitor and validate:** Compare responses (shadow mode), track error rates.
  7) **Cutover:** Switch writes to new service; deprecate legacy code.
  8) **Repeat:** Migrate next slice (orders, users, payments).
  9) **Decommission:** When all features migrated, shut down monolith.

## Trade-offs / Costs (and their mitigation)
| Trade-off | Mitigation |
|-----------|-----------|
| Long migration (months/years) | Set realistic timeline; deliver value incrementally; avoid perfectionism |
| Dual-write complexity | Use [Change Data Capture](../databases/change-data-capture.md) or event-driven sync |
| Data consistency issues | Accept eventual consistency; use [Saga](saga-pattern.md) for critical flows |
| Operational overhead (two systems) | Automate deployment; use feature flags to toggle routing |
| Risk of "strangler never finishes" | Set deadlines; track migration progress; celebrate milestones |

## Failure modes / Edge cases
1. **Dual-write drift:** Legacy and new service DBs diverge.
   - *Mitigation:* Use [Outbox pattern](outbox-pattern.md) or CDC; [reconcile](../operations/data-reconciliation.md) periodically.
2. **Routing complexity explodes:** 100s of if-else rules in gateway.
   - *Mitigation:* Use declarative routing (config-driven); refactor incrementally.
3. **Hidden dependencies:** New service calls legacy, creating tight coupling.
   - *Mitigation:* Use [Anti-Corruption Layer](anti-corruption-layer.md) to isolate.
4. **Migration stalls:** Team loses momentum after initial slice.
   - *Mitigation:* Dedicate resources; track progress publicly; celebrate wins.
5. **Legacy becomes zombie:** Never fully decommissioned.
   - *Mitigation:* Set hard deadline; budget for final cleanup; measure legacy usage.

## Alternatives
- **Big bang rewrite:** Replace entire system at once (high risk).
- **Feature freeze + rewrite:** Stop new features, rewrite, then cutover (business impact).
- **Parallel run:** Run legacy + new in parallel, switch after validation (complex).
- **Lift and shift:** Migrate to cloud without refactoring (doesn't modernize).

## Phases
### Phase 1: Preparation
- Deploy proxy/gateway.
- Identify first migration slice.
- Establish monitoring and observability.

### Phase 2: Build New Service
- Develop microservice.
- Migrate data schema (if needed).
- Test thoroughly (unit, integration, contract).

### Phase 3: Dual-Write (if needed)
- Write to both systems temporarily.
- Validate consistency.

### Phase 4: Route Traffic
- Shadow mode: route reads to new, compare with legacy.
- Cutover: route production traffic to new service.
- Rollback plan ready.

### Phase 5: Decommission
- Stop dual-writes.
- Delete legacy code.
- Migrate next slice.

## Combinations
Strangler Fig is **almost always used with:**
- **[API Gateway](../system-design/api-gateway.md):** Routes traffic to new vs legacy.
- **[Anti-Corruption Layer](anti-corruption-layer.md):** Protects new services from legacy models.
- **[Change Data Capture](../databases/change-data-capture.md):** Sync data between systems.
- **[Observability](../operations/observability-basics.md):** Monitor both systems during migration.
- **[Feature flags](../quality/software-quality-assurance.md):** Toggle routing dynamically.

**Typical combination:**
- **Legacy migration scenario:** Strangler Fig + API Gateway + ACL + Observability + Feature Flags

## Prerequisites
- Understanding of [Microservices design](microservices-design-basics.md).
- Familiarity with [API Gateway](../system-design/api-gateway.md) and routing.
- Knowledge of [Data migration](../databases/data-modeling-basics.md) and consistency.

## Related topics
- [API Gateway](../system-design/api-gateway.md): Routing layer for strangler.
- [Anti-Corruption Layer](anti-corruption-layer.md): Isolate new services from legacy.
- [Dual-write pattern](dual-write-pattern.md): Temporary sync during migration.
- [Change Data Capture](../databases/change-data-capture.md): Alternative to dual-write.
- [Microservices design](microservices-design-basics.md): Target architecture.

## Real-world examples
1. **Shopify:** Migrated Rails monolith to microservices over 5+ years using strangler.
2. **SoundCloud:** Used strangler to extract user/playback services from monolith.
3. **Martin Fowler's original article (2004):** Coined the term based on real project.

## Checklist (self-test)
- [ ] I can explain why big bang rewrites are risky.
- [ ] I know how to set up routing proxy/gateway.
- [ ] I understand dual-write challenges and alternatives.
- [ ] I can plan migration in slices (prioritize by value/risk).
- [ ] I can monitor both legacy and new services during migration.

## Reminders / Key takeaways
- **Incremental > Big Bang:** Reduce risk, deliver value continuously.
- Use [API Gateway](../system-design/api-gateway.md) for routing.
- Use [ACL](anti-corruption-layer.md) to protect new services from legacy models.
- Plan data migration carefully (dual-write, CDC, [reconciliation](../operations/data-reconciliation.md)).
- Set **migration deadline** to avoid zombie legacy.

## Trap questions (with answers)
### Q1: Should I migrate the entire database schema at once?
**A:** **No**. Migrate data **incrementally** per service slice. Use:
1. **Dual-write:** Temporarily write to both legacy + new DB.
2. **[Change Data Capture](../databases/change-data-capture.md):** Stream changes from legacy to new DB.
3. **Batch migration:** Copy historical data offline; sync deltas.
Avoid "one big migration" — too risky. Accept eventual consistency during transition.

### Q2: How long should a strangler migration take?
**A:** **Depends on monolith size and team capacity**. Typical: 1-3 years for large monoliths. Guidelines:
- Set **realistic timeline** (not "6 months" for 10-year-old monolith).
- Deliver value incrementally (new features in new services).
- Track progress (e.g., "60% of traffic to new services").
- Set **hard deadline** for final cutover (avoid zombie legacy).

### Q3: What if the new service fails during migration?
**A:** **Have rollback plan**. With strangler, you can:
1. Route traffic **back to legacy** instantly (toggle gateway routing).
2. Run both systems in parallel (fallback to legacy).
3. Monitor error rates and rollback automatically (circuit breaker).
This is strangler's key advantage: **low-risk rollback** vs big bang (no rollback).

### Q4: Can I add new features to the legacy system during migration?
**A:** **Generally avoid it**. Adding features to legacy:
- Increases migration effort (more to migrate later).
- Couples team to legacy tech stack.
**Better:** Build new features in **new microservices** from day 1. Use strangler proxy to route new endpoints to new services. This delivers value while migrating.

### Q5: How do I handle shared data (e.g., users table used by multiple features)?
**A:** **Options:**
1. **[Database per Service](database-per-service.md):** Extract user-service, own users table, expose via API.
2. **Shared DB temporarily:** Allow new services to read legacy DB (via ACL) during migration; migrate later.
3. **Event-driven sync:** Legacy publishes user events; new services consume and build local view.
Choose based on coupling tolerance. Ideally, extract shared data into dedicated service early. See [Bounded Contexts](bounded-contexts.md).
