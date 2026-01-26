---
id: anti-corruption-layer
title: "Anti-Corruption Layer (ACL)"
type: pattern
status: learning
importance: 70
difficulty: medium
tags: [architecture, domain-driven-design, integration, legacy]
canonical: true
aliases: [acl, adapter layer, translation layer]
created_at: 2026-01-26
updated_at: 2026-01-26
---

# Anti-Corruption Layer (ACL)

## TL;DR (BLUF)
- A translation layer that protects your domain model from external/legacy systems with incompatible models.
- Use it when integrating with third parties, legacy systems, or poorly designed APIs.
- Trade-off: added complexity and translation overhead.

## Definition
**What it is:** A layer (adapter, facade, translator) that sits between your clean domain model and an external system, translating between models and preventing external concepts from "leaking" into your core domain.

**Key terms:** adapter, facade, translation, domain protection, boundary, integration layer, impedance mismatch.

## Why it matters
- Preserves domain integrity: external models don't pollute your core.
- Enables gradual migration from legacy systems.
- Reduces coupling to third-party APIs (if API changes, only ACL changes).
- Enforces ubiquitous language within your bounded context (DDD).

## Scope & Non-goals
**In scope:** model translation, validation, normalization, error handling, protocol adaptation.

**Out of scope / NOT solved by this:**
- Business logic (belongs in domain layer)
- Data persistence (use repositories)
- API Gateway concerns (routing, rate limiting) → [API Gateway](../system-design/api-gateway.md)

## Mental model / Intuition
- Like a translator at a business meeting: converts foreign language/concepts to yours so your team understands.
- In systems: legacy "Customer" (ID, Name, Addr1, Addr2) → domain `Customer` (ID, Name, Address value object).

## Decision rules (When to use / When not to use)
### Use it when
- Integrating with legacy systems with poor/inconsistent models.
- Third-party APIs use different terminology or structure.
- External model is unstable or frequently changes.
- You want to isolate domain from integration details.
- Migrating from monolith to microservices ([Strangler Fig](strangler-fig.md)).

### Avoid it when
- External system aligns well with your domain (rare).
- Integration is trivial (simple data pass-through).
- Overhead of translation outweighs benefits.
- You own both systems and can align models directly.

## How I would use it (practical)
- **Context:** E-commerce system integrating with legacy ERP that uses flat, denormalized data.
- **Steps:**
  1) **Define domain model (clean):**
     ```go
     type Order struct {
         ID       string
         Customer Customer
         Items    []OrderItem
         Address  ShippingAddress // Value object
         Status   OrderStatus     // Enum
     }
     ```
  2) **External ERP model (messy):**
     ```xml
     <ORDER>
       <ORDER_ID>12345</ORDER_ID>
       <CUST_NAME>John Doe</CUST_NAME>
       <CUST_ID>C001</CUST_ID>
       <SHIP_ADDR1>123 Main St</SHIP_ADDR1>
       <SHIP_ADDR2>Apt 4B</SHIP_ADDR2>
       <SHIP_CITY>NYC</SHIP_CITY>
       <STATUS_CODE>2</STATUS_CODE> <!-- Magic number -->
     </ORDER>
     ```
  3) **ACL (adapter):**
     ```go
     type ERPAdapter struct {}
     
     func (a *ERPAdapter) FetchOrder(erpOrderID string) (*Order, error) {
         erpOrder := erpClient.GetOrder(erpOrderID) // XML mess
         
         // Translate to domain model
         return &Order{
             ID: erpOrder.OrderID,
             Customer: Customer{
                 ID:   erpOrder.CustomerID,
                 Name: erpOrder.CustomerName,
             },
             Address: ShippingAddress{
                 Street: erpOrder.ShipAddr1 + " " + erpOrder.ShipAddr2,
                 City:   erpOrder.ShipCity,
             },
             Status: translateStatus(erpOrder.StatusCode), // 2 → "Shipped"
         }, nil
     }
     
     func translateStatus(code int) OrderStatus {
         switch code {
         case 1: return OrderStatusPending
         case 2: return OrderStatusShipped
         default: return OrderStatusUnknown
         }
     }
     ```
  4) **Domain layer only sees clean `Order` model**, never touches ERP XML.
  5) If ERP changes, only ACL changes (domain untouched).

## Trade-offs / Costs (and their mitigation)
| Trade-off | Mitigation |
|-----------|-----------|
| Translation overhead (CPU, latency) | Cache translated objects; use efficient serialization |
| Impedance mismatch (concepts don't map 1:1) | Accept data loss; document unmappable fields |
| ACL becomes complex | Keep it thin (translation only); extract domain logic to services |
| Duplication (domain model + external model) | Acceptable cost for domain isolation; use code generation if possible |
| Versioning both models | Version ACL; use adapter per external API version |

## Failure modes / Edge cases
1. **ACL becomes a dumping ground:** Business logic leaks into ACL.
   - *Mitigation:* Enforce policy: ACL is translation only; logic stays in domain.
2. **External model changes break ACL:** New fields, removed fields.
   - *Mitigation:* Defensive parsing (ignore unknown fields); monitor integration tests.
3. **Translation loses semantic meaning:** External "status=3" unclear.
   - *Mitigation:* Document mappings; default to safe values (e.g., Unknown).
4. **Bidirectional translation inconsistency:** Domain → External → Domain ≠ original.
   - *Mitigation:* Test round-trip conversions; accept lossy translations if acceptable.
5. **ACL bypassed:** Developers call external system directly.
   - *Mitigation:* Encapsulate external client in ACL; make it private.

## Alternatives
- **Shared kernel:** Align models across systems (requires ownership).
- **Conformist:** Adopt external model as-is (pollutes domain).
- **Open Host Service:** Publish well-defined API for others to consume.
- **Published Language:** Standardize on common model (JSON Schema, Protocol Buffers).

## Combinations
Anti-Corruption Layer is **almost always used with:**
- **[Strangler Fig](strangler-fig.md):** Migrate from legacy by wrapping with ACL.
- **[Hexagonal Architecture](hexagonal-architecture.md):** ACL is an adapter (port/adapter pattern).
- **[Domain-Driven Design](domain-driven-design.md):** Protects bounded context.
- **[Circuit Breaker](../operations/circuit-breaker.md):** Protect against flaky external systems.
- **[Timeout](../operations/timeouts.md):** Prevent slow external calls from blocking.

**Typical combination:**
- **Legacy integration scenario:** ACL + Strangler Fig + Circuit Breaker + Timeout + Observability

## Prerequisites
- Understanding of [Domain-Driven Design](domain-driven-design.md) and [Bounded Contexts](bounded-contexts.md).
- Familiarity with [Hexagonal Architecture](hexagonal-architecture.md) (ports/adapters).
- Knowledge of integration patterns and API design.

## Related topics
- [Strangler Fig](strangler-fig.md): Migrate legacy incrementally.
- [Hexagonal Architecture](hexagonal-architecture.md): ACL is an adapter.
- [Bounded Contexts](bounded-contexts.md): ACL protects context boundaries.
- [Domain-Driven Design](domain-driven-design.md): Foundation for ACL.
- [Circuit Breaker](../operations/circuit-breaker.md): Resilience for external calls.

## Real-world examples
1. **E-commerce + legacy ERP:** ACL translates ERP's flat CSV to domain `Order` objects.
2. **SaaS integrating Salesforce:** ACL adapts Salesforce's `Account` to internal `Customer`.
3. **Payment gateway integration:** ACL normalizes Stripe/PayPal responses to domain `Payment`.

## Checklist (self-test)
- [ ] I can identify when external models threaten domain integrity.
- [ ] I know how to design translation logic (defensive parsing, defaults).
- [ ] I can explain the difference between ACL and simple adapter.
- [ ] I understand how ACL fits in hexagonal architecture (adapter layer).
- [ ] I can monitor ACL health (translation errors, external API changes).

## Reminders / Key takeaways
- ACL **protects your domain** from external pollution.
- Keep ACL **thin**: translation only, no business logic.
- Use with [Strangler Fig](strangler-fig.md) to migrate legacy.
- Always combine with [Circuit Breaker](../operations/circuit-breaker.md) and [Timeout](../operations/timeouts.md) for external calls.

## Trap questions (with answers)
### Q1: Can I put validation logic in the ACL?
**A:** **Yes, but only for external data**. ACL should validate that external data is well-formed (non-null fields, valid enums) before translation. However, **domain validation** (business rules like "order total > 0") belongs in the domain layer, not ACL. ACL validates **data shape**, domain validates **business invariants**.

### Q2: What's the difference between ACL and a simple adapter?
**A:** **ACL is a specific type of adapter** with a **protective intent**: it prevents external models from corrupting your domain. A generic adapter might just translate for convenience. ACL is motivated by **domain integrity** (DDD concept), while a simple adapter might just adapt protocols (HTTP → gRPC). If you care about bounded contexts and ubiquitous language, you need ACL.

### Q3: Should I create an ACL for every external integration?
**A:** **Not always**. Use ACL when:
1. External model is messy/unstable.
2. External concepts don't align with your domain.
3. You want to isolate domain from integration details.

Skip ACL when:
1. External API is well-designed and aligns with your domain.
2. Integration is trivial (fetching config, health checks).
3. Overhead isn't justified (small project, tight deadline).

### Q4: Can ACL handle bidirectional translation (read + write)?
**A:** **Yes**. ACL translates in both directions:
- **Inbound:** External → Domain (when reading from external system).
- **Outbound:** Domain → External (when writing to external system).

Example: `fetchOrder()` translates ERP → Domain; `syncOrder()` translates Domain → ERP. Be careful of **lossy translations**: round-trip may not preserve all data.

### Q5: How do I test an ACL?
**A:** 
1. **Unit tests:** Test translation logic (external DTO → domain model).
2. **Contract tests:** Ensure external API hasn't changed (detect breaking changes).
3. **Integration tests:** Test full flow (call external system via ACL).
4. **Round-trip tests:** Domain → External → Domain (validate consistency).
Use test fixtures for external responses to avoid flaky tests. See [Testing pyramid](../quality/testing-pyramid.md).
