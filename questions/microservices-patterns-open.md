# Open Questions: Microservices Patterns (Real-World Focus)

## Instructions
These are interview-style open-ended questions focused on:
- **Real problems** each pattern solves
- **Consequences** of not using the pattern
- **Trade-offs** and problems introduced
- **Real examples** (including failure scenarios)
- **Pattern combinations** (what goes together)

Answer each question in **3-5 minutes** (interview format). Focus on BLUF (Bottom Line Up Front), then details.

---

## API Gateway

### Q1: Explain a real scenario where NOT having an API Gateway caused problems. What would the API Gateway have prevented?

**Expected coverage:**
- Problem: clients calling services directly, auth scattered, no rate limiting
- Consequence: security breach OR cascading failure OR client coupling
- How gateway solves: centralized auth, rate limiting, routing isolation
- Trade-off: added latency, single point of failure

---

### Q2: You're designing a SaaS API with free and premium tiers. How would you use API Gateway to enforce this? What problems might you encounter?

**Expected coverage:**
- Rate limiting per tier (free: 100 req/min, premium: 10k)
- API key-based routing and quotas
- Problems: shared IP (NAT), thundering herd on limit reset
- Mitigation: use API key not IP, sliding window algorithm
- Combine with: rate limiting, observability

---

## BFF (Backend for Frontend)

### Q3: You have mobile and web clients calling the same backend API. Users complain mobile is slow and uses too much data. How do you solve this? What pattern would you use and why?

**Expected coverage:**
- Problem: shared API forces lowest common denominator
- Solution: BFF (mobile-BFF optimized for slow networks)
- Mobile-BFF: minimal payload, aggressive caching, optimized images
- Trade-off: code duplication, operational overhead
- Combine with: API Gateway (routes to BFFs), API Composition

---

### Q4: When should you NOT use a BFF? Give a concrete example.

**Expected coverage:**
- Single client type (only web OR only mobile)
- Trivial aggregation (client can fetch directly)
- Operational overhead too high for small team
- Example: internal admin tool (one client, simple CRUD)

---

## Circuit Breaker + Timeout + Retry (Trinity)

### Q5: A payment service calls a third-party fraud detection API. The API is slow (5s responses) and sometimes times out. What patterns would you combine to protect your system? Explain the sequence.

**Expected coverage:**
- Timeout: 2s max
- Retry: 3 attempts with exponential backoff (transient failures)
- Circuit breaker: open after 5 consecutive failures
- Fallback: approve payment (graceful degradation) OR queue for manual review
- Why all three: timeout detects slow, retry handles transient, CB stops cascades
- Combine with: Bulkhead (isolate fraud API pool), Observability

---

### Q6: What happens if you have retries but NO timeout? Give a real failure scenario.

**Expected coverage:**
- Slow downstream (10s response)
- Retries keep waiting 10s each → 30s total
- Connection pool saturates, entire service hangs
- Cascading failure to upstreams
- Fix: timeout + retry (retry only fast failures)

---

## Saga Pattern

### Q7: Design a checkout flow (reserve inventory → charge payment → create order). One step fails. How do you handle it? What pattern and why?

**Expected coverage:**
- Saga pattern (orchestrated or choreographed)
- Steps: reserve → charge → create
- Failure: payment fails → compensate (release inventory)
- Orchestrated: central orchestrator tracks state
- Choreographed: services listen to events, react
- Trade-off: eventual consistency, complex compensation logic
- Combine with: Outbox (reliable events), Idempotency (safe retries)

---

### Q8: What's the biggest risk of Saga pattern? Give an example of a Saga gone wrong.

**Expected coverage:**
- Risk: compensation failure (can't undo)
- Example: charge succeeded, inventory reservation deleted (race condition)
- Result: customer charged but no order
- Mitigation: idempotent compensations, timeout limits, manual reconciliation
- Alternative: if you need ACID, reconsider service boundaries

---

## CQRS

### Q9: You have a news platform. Writes are simple (publish article), but reads are complex (search, filters, recommendations). How would CQRS help? What problems does it introduce?

**Expected coverage:**
- Write model: article-service (Postgres, normalized)
- Read models: search-service (Elasticsearch), recommendation-service (graph DB)
- Writes publish events → read models subscribe and update
- Benefit: optimize each side independently
- Problem: eventual consistency (search lag), dual-write complexity
- Combine with: Outbox (reliable events), Observability (monitor lag)

---

### Q10: When is CQRS overkill? Give an example where you should NOT use it.

**Expected coverage:**
- Simple CRUD app (admin panel, internal tool)
- Reads and writes have same model
- Team too small to manage multiple models
- Example: task management app (todos) → standard REST API is fine

---

## Strangler Fig

### Q11: Your company has a 10-year-old monolith. Management wants to "rewrite it in 6 months." Why is this dangerous? What would you recommend instead?

**Expected coverage:**
- Risk: big-bang rewrite fails (second system syndrome)
- Business stops during rewrite (no new features)
- Unknown unknowns in legacy (lose domain knowledge)
- Alternative: Strangler Fig (incremental migration)
- Steps: gateway → extract one service at a time → route traffic → decommission
- Timeline: realistic (1-3 years for large monolith)
- Combine with: API Gateway, ACL, Observability

---

### Q12: During Strangler Fig migration, you have data in both legacy DB and new service DB. How do you keep them consistent? What problems can occur?

**Expected coverage:**
- Dual-write: write to both DBs temporarily
- Problem: writes can diverge (one succeeds, one fails)
- Solutions: Outbox pattern, Change Data Capture, batch reconciliation
- Accept eventual consistency during migration
- Monitor: data drift, reconciliation lag

---

## Anti-Corruption Layer (ACL)

### Q13: You're integrating with a legacy ERP that uses magic numbers (status=2 means "Shipped"). How do you protect your domain model? What pattern and why?

**Expected coverage:**
- ACL translates ERP model → domain model
- Example: status=2 → OrderStatus.Shipped (enum)
- ACL is thin: translation only, no business logic
- Benefits: domain isolated, ERP changes don't propagate
- Combine with: Hexagonal Architecture (ACL is adapter), Circuit Breaker

---

### Q14: Can you put validation logic in the ACL? Where's the line between ACL validation and domain validation?

**Expected coverage:**
- ACL validates data shape (non-null, valid enums, format)
- Domain validates business rules (order total > 0, stock available)
- ACL: "Is this parseable?"
- Domain: "Is this allowed?"
- If unclear, err on side of domain validation

---

## Outbox + Idempotency (Event Reliability)

### Q15: Explain the dual-write problem. How does Outbox pattern solve it? What happens if you skip Outbox?

**Expected coverage:**
- Dual-write problem: DB write succeeds, event publish fails → lost event
- Outbox: write to DB + outbox table in same transaction
- Relay process publishes from outbox → Kafka (at-least-once)
- Consumer uses idempotency key to dedupe
- Skip Outbox: risk lost events, inconsistent state

---

### Q16: You publish the same event twice (network retry). Downstream service processes it twice and charges customer twice. How do you prevent this?

**Expected coverage:**
- Idempotency: use idempotency key (event ID, request ID)
- Consumer stores processed keys in DB
- Before processing: check if key exists
- If exists: skip (already processed)
- If new: process + store key
- Combine with: Outbox (reliable publishing), Retry (safe to retry)

---

## Service Discovery

### Q17: You deploy services on Kubernetes with autoscaling. How do services find each other? Why can't you use hardcoded IPs?

**Expected coverage:**
- Service Discovery (Kubernetes DNS)
- Services call by name: `http://user-service`
- Kubernetes resolves to available pod IPs
- Autoscaling: pods come/go, IPs change
- Hardcoded IPs break on deploy/scale
- Combine with: Health checks (readiness probes), Observability

---

### Q18: What happens if the service registry goes down? How do you mitigate this?

**Expected coverage:**
- Problem: new instances can't be discovered
- Existing connections may work (cached)
- Mitigation: cache last-known instances locally
- Use highly available registry (Kubernetes DNS is distributed)
- Set long cache TTLs as fallback (trade staleness for availability)

---

## Rate Limiting

### Q19: You have a public API. A malicious user rotates IPs to bypass rate limits. What do you do?

**Expected coverage:**
- Problem: IP-based rate limiting fails with IP rotation
- Solution: rate limit by API key or user ID (requires auth)
- For unauthenticated: CAPTCHA, fingerprinting, WAF
- Combine with: API Gateway (centralized enforcement), Observability (detect patterns)

---

### Q20: Should you count 5xx errors (server errors) against the user's rate limit quota? Why or why not?

**Expected coverage:**
- No: don't punish users for your failures
- Count 4xx (client errors): prevents probing attacks
- Exception: 429 itself should NOT consume quota (allows graceful retry)
- Monitor both: user quota usage + server error rate

---

## Graceful Degradation

### Q21: Amazon's product page loads but recommendations are missing. What pattern is this? Why not just return 503?

**Expected coverage:**
- Graceful Degradation: partial response (critical + optional)
- Critical: product details (must work)
- Optional: recommendations (nice-to-have)
- If recommendations fail: omit section, page still loads
- UX: "something" > "nothing"
- Combine with: Circuit Breaker (detect failure), Timeout, Caching (fallback)

---

### Q22: When should you NOT use graceful degradation? Give an example where failing fast is better.

**Expected coverage:**
- Critical paths: auth, payment, order creation
- Example: payment service down → DON'T fake payment success
- Fail fast with 503: clear error, user can retry
- Safety-critical: healthcare, aviation (no degradation)

---

## Database per Service

### Q23: You need to show "Order Details" which includes user name. User-service owns users table, order-service owns orders table. How do you get the data? What are the options and trade-offs?

**Expected coverage:**
- Option 1: API call (order-service → user-service)
  - Pro: real-time, accurate
  - Con: latency, coupling
- Option 2: Cache userName in order-service (eventual consistency)
  - Pro: fast, no runtime call
  - Con: stale data (user changes name)
- Option 3: CQRS read model (aggregate both)
  - Pro: optimized for query
  - Con: eventual consistency, complexity
- Recommendation: Cache + event sync for this case

---

### Q24: What's the biggest mistake teams make when adopting "database per service"? How do you avoid it?

**Expected coverage:**
- Mistake: services call each other's DBs directly (defeats the purpose)
- Result: tight coupling, broken encapsulation
- Prevention: network policies (DB firewall), code reviews
- Enforce: API-only access, no shared DB credentials
- Combine with: Observability (detect direct DB calls)

---

## Pattern Combinations

### Q25: You're building an e-commerce checkout. Design the flow using Saga + Outbox + Idempotency. Explain why you need all three.

**Expected coverage:**
- Saga: coordinate steps (reserve → charge → create)
- Outbox: publish events reliably (no lost events)
- Idempotency: safe retries (no double charge)
- Flow: order-service writes order + event to outbox (atomic)
- Relay publishes event → payment-service charges
- Payment uses idempotency key (request ID)
- If payment fails: saga compensates (release reservation)

---

### Q26: You have a BFF aggregating data from 5 services. One service is slow (3s response). What patterns would you combine to handle this? Explain the sequence.

**Expected coverage:**
- Timeout: 500ms per service
- Circuit Breaker: fail fast if service is unhealthy
- Graceful Degradation: return partial data (omit slow service)
- Parallel calls: call all 5 in parallel (not sequential)
- Caching: serve stale data if timeout
- Observability: track which service is slow
- Result: page loads in < 1s with partial data

---

## Meta-Questions (Pattern Selection)

### Q27: Your team wants to use every pattern in this list. Why is this a bad idea? How do you decide which patterns to use?

**Expected coverage:**
- Over-engineering: patterns add complexity
- Decision criteria:
  1. What problem are we solving?
  2. What's the simplest solution?
  3. What's the cost (complexity, latency, operational overhead)?
- Start small: add patterns when problems emerge
- Example: Don't use CQRS if simple REST API works
- Balance: solve real problems, avoid premature optimization

---

### Q28: Rank these in order of importance for a production microservices system: (1) Timeout, (2) API versioning, (3) CQRS, (4) Circuit Breaker, (5) Observability. Justify your ranking.

**Expected coverage:**
- Ranking (typical):
  1. Observability (blind without it)
  2. Timeout (prevents hangs)
  3. Circuit Breaker (prevents cascades)
  4. API versioning (enables evolution)
  5. CQRS (only if needed)
- Justification: resilience patterns are foundational; CQRS is situational
- Context matters: ranking may change based on system

---

## Evaluation Rubric (for each answer)
1. **Problem identification** (did they explain the real problem?)
2. **Pattern selection** (did they choose the right pattern?)
3. **Trade-offs** (did they mention costs and alternatives?)
4. **Real example** (concrete scenario, not abstract)
5. **Combinations** (did they mention related patterns?)
6. **Failure modes** (what can go wrong?)

---

## Next Steps
After answering these questions:
1. Record answers and timestamp.
2. Self-evaluate using rubric (0-5 per criterion).
3. Identify gaps (which patterns need more study?).
4. Log results in `practices/logs/YYYY-MM-DD.microservices-patterns-open.md`.
5. Update `practices/next.md` with focus areas.
