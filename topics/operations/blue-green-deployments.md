---
id: blue-green-deployments
title: "Blue/Green Deployments"
type: pattern
status: learning
importance: 75
difficulty: medium
tags: [deployment, reliability, zero-downtime, operations]
canonical: true
aliases: [blue-green, red-black-deployment]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Blue/Green Deployments

## TL;DR (BLUF)
- Blue/green deployment maintains two identical production environments (blue = current, green = new); switch traffic instantly between them.
- Use for zero-downtime deployments with instant rollback capability.
- Key trade-off: infrastructure cost (2x environments) vs. deployment safety and speed.

## Definition
**What it is:** A deployment strategy that uses two identical production environments where only one serves live traffic at a time, allowing instant switching between versions.  
**Key terms:** blue environment (current production), green environment (new version), cutover (traffic switch), rollback, zero-downtime deployment, load balancer, [DNS](dns.md) switching.

## Why it matters
- **Zero downtime:** Users experience no service interruption during deployments.
- **Instant rollback:** If issues arise, switch back to blue environment in seconds.
- **Safe testing:** Validate green environment with production load before cutover.
- **Simplified process:** Deployment is just a traffic switch (no complex migration).
- **Interview relevance:** Common pattern in system design discussions about deployment strategies.

## Scope & Non-goals
**In scope:**
- Blue/green deployment mechanics (two environments, traffic switching).
- When to use blue/green vs. other strategies.
- Cutover mechanisms (load balancer, [DNS](dns.md), feature flags).

**Out of scope / NOT solved by this:**
- Database migrations (separate challenge; requires forward-compatible schemas).
- Cost optimization strategies (blue/green is expensive).
- Specific cloud platform implementations (AWS, GCP, Azure—though concepts apply).

## Mental model / Intuition
Think of blue/green as **two identical stages with a spotlight**:
- **Blue stage:** Currently lit (serving production traffic).
- **Green stage:** Dark (new version ready but not serving traffic).
- **Deployment:** Flip the spotlight from blue to green (instant cutover).
- **Rollback:** Flip spotlight back to blue (instant revert).

No one notices the switch because both stages are identical infrastructure.

## Decision rules (When to use / When not to use)
### Use it when
- Zero downtime is critical (customer-facing services, SLAs).
- Rollback speed is essential (ability to revert in seconds).
- Deployment validation is needed (smoke tests on green before cutover).
- Infrastructure cost is acceptable (2x resources during deployment).

### Avoid it when
- Infrastructure cost is prohibitive (startup, small-scale services).
- Database migrations are complex (schema changes require careful planning).
- Canary deployments are better suited (gradual rollout preferred).
- Resources are constrained (can't afford 2x environments).

## How I would use it (practical)
- **Context:** Deploying a Go API service on Kubernetes with PostgreSQL database.
- **Steps:**
  1. **Blue environment (current):**
     - Deployment: `api-blue` with 10 pods serving traffic.
     - Service: `api-service` routes to `api-blue`.
     - Database: `production-db` (shared between blue/green).
  2. **Deploy green environment:**
     - Create new deployment: `api-green` with 10 pods.
     - Run smoke tests against green (health checks, critical endpoints).
     - Monitor green for 5-10 minutes (check metrics, logs).
  3. **Cutover (traffic switch):**
     - Update service selector: `api-service` now routes to `api-green`.
     - Traffic switches instantly (users now hit green).
     - Monitor green closely (error rate, latency).
  4. **Post-cutover:**
     - If issues: revert service to `api-blue` (instant rollback).
     - If stable: keep `api-blue` running for 24 hours (fallback option).
     - After validation: tear down `api-blue` (save costs).
- **What success looks like:** Deployment in <10 minutes; zero user-facing downtime; rollback in <30 seconds if needed.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Zero downtime (seamless user experience).
  - Instant rollback (switch back in seconds).
  - Safe validation (test green before cutover).
  - Simple cutover (just change routing).
- **Cons / Risks:**
  - High infrastructure cost (2x environments during deployment).
  - Database complexity (schema changes must be forward-compatible).
  - Stateful services are challenging (session state, in-progress work).
  - Resource waste (blue environment idle after cutover).

### Alternatives
- **[Canary deployments](canary-deployments.md):** Gradual rollout to subset of traffic (lower risk, lower cost).
- **Rolling deployments:** Replace instances one-by-one (zero extra cost but slower rollback).
- **Recreate deployment:** Stop old, start new (simple but has downtime).

### How to choose
- **Zero downtime + instant rollback required:** Blue/green.
- **Cost-sensitive or gradual validation preferred:** Canary.
- **Simple services, downtime acceptable:** Rolling or recreate.
- **Complex state or database migrations:** Consider feature flags + rolling.

## Failure modes & Pitfalls
- **Database incompatibility:** Green uses new schema; blue can't read it (breaks rollback).
- **Shared state issues:** Sessions, caches, queues cause inconsistencies between blue/green.
- **No smoke tests on green:** Switch to broken environment; discover issues in production.
- **Premature blue teardown:** Delete blue too soon; can't rollback when issues appear later.
- **[DNS](dns.md) cutover delays:** DNS caching causes gradual switch instead of instant (use load balancer instead).

## Observability (How to detect issues)
- **Metrics:**
  - Error rate (monitor green during and after cutover; compare to blue baseline).
  - Latency (P95/P99; flag spikes in green).
  - Traffic distribution (ensure 100% to green after cutover).
  - Resource utilization (CPU, memory on both environments).
- **Logs:** Compare error logs between blue and green (look for new errors).
- **Traces:** Validate green handles requests correctly (no timeout spikes).
- **Alerts:** Error rate spike, latency increase, health check failures on green.

## Implementation notes
- **Checklist**
  - [ ] Ensure database schema is forward-compatible (green and blue can both read/write).
  - [ ] Deploy green environment (identical to blue but with new code).
  - [ ] Run smoke tests on green (health checks, critical endpoints).
  - [ ] Monitor green for 5-10 minutes before cutover.
  - [ ] Switch traffic from blue to green (load balancer or service selector).
  - [ ] Monitor green closely post-cutover (error rate, latency, logs).
  - [ ] Keep blue running for fallback period (24 hours recommended).
  - [ ] Tear down blue after validation (save costs).

- **Security / Compliance notes**
  - Ensure both environments have identical security posture (configs, secrets, IAM roles).
  - Audit cutover events (who switched traffic, when).

- **Performance notes**
  - Warm up green environment before cutover (populate caches, pre-load data).
  - Use connection draining on blue (graceful shutdown of in-flight requests).

- **Operational notes**
  - Automate cutover (scripts, CI/CD) to reduce human error.
  - Document rollback procedure (must be executable under pressure).

## Mini example (Kubernetes)
```yaml
# Blue deployment (current production)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-blue
spec:
  replicas: 10
  selector:
    matchLabels:
      app: api
      version: blue
  template:
    metadata:
      labels:
        app: api
        version: blue
    spec:
      containers:
      - name: api
        image: myapp:v1.0

---
# Green deployment (new version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-green
spec:
  replicas: 10
  selector:
    matchLabels:
      app: api
      version: green
  template:
    metadata:
      labels:
        app: api
        version: green
    spec:
      containers:
      - name: api
        image: myapp:v2.0

---
# Service (controls traffic routing)
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api
    version: blue  # Switch to "green" for cutover
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080

# Cutover: kubectl patch svc api-service -p '{"spec":{"selector":{"version":"green"}}}'
# Rollback: kubectl patch svc api-service -p '{"spec":{"selector":{"version":"blue"}}}'
```

## Common anti-patterns
- **Anti-pattern:** No smoke tests on green before cutover.
  - **Why it's bad:** Switch to broken environment; discover issues in production.
  - **Better approach:** Run smoke tests, monitor green for 5-10 minutes before cutover.

- **Anti-pattern:** Database schema incompatible between blue and green.
  - **Why it's bad:** Rollback to blue fails (can't read green's schema).
  - **Better approach:** Use forward-compatible schemas (blue can read green's data, even if it doesn't use new fields).

- **Anti-pattern:** Instant teardown of blue after cutover.
  - **Why it's bad:** Issues may appear hours later; can't rollback.
  - **Better approach:** Keep blue running for 24 hours (or longer for critical systems).

- **Anti-pattern:** Using [DNS](dns.md) for cutover.
  - **Why it's bad:** DNS caching causes slow, unpredictable switch (not instant).
  - **Better approach:** Use load balancer or service mesh for instant traffic switching.

## Interview readiness
### "Explain it like I'm teaching"
Blue/green deployment uses two identical production environments. Blue is the current version serving traffic, and green is the new version that's deployed but not yet live. When green is ready, you switch traffic from blue to green instantly—typically by changing a load balancer configuration or Kubernetes service selector. If issues arise, you switch back to blue in seconds. The main benefits are zero downtime and instant rollback. The main cost is running two full environments during deployment. It's ideal when downtime is unacceptable and rollback speed is critical.

### Trap questions (with answers)
1) **Q:** How do you handle database migrations in blue/green deployments?
   - **A:** Use forward-compatible schemas. Green's schema changes must allow blue to still read/write (e.g., add nullable columns, don't remove columns). Run migrations before deploying green. For complex migrations, use expand/contract pattern: expand schema (add new fields), deploy green (uses new fields), contract schema (remove old fields) after blue is retired.

2) **Q:** What's the difference between blue/green and canary deployments?
   - **A:** Blue/green switches 100% of traffic from old to new version instantly. Canary routes a small percentage to the new version gradually (5%, 10%, 50%, 100%). Blue/green has faster cutover and rollback but higher cost (2x infrastructure). Canary has lower cost and risk (gradual validation) but slower rollout.

3) **Q:** How do you handle stateful services (sessions, WebSockets) in blue/green?
   - **A:** Options: (1) Use sticky sessions (route user to same environment until session expires). (2) External session store (Redis) shared by blue and green. (3) Graceful connection draining (let existing connections finish on blue before switching). (4) Client reconnects (WebSockets reconnect to green after cutover).

4) **Q:** Isn't blue/green wasteful (2x infrastructure cost)?
   - **A:** Yes, it's expensive. Blue and green both run during deployment. After cutover, blue is idle (but kept for rollback). For cost-sensitive scenarios, use canary or rolling deployments instead. Blue/green is justified when zero downtime and instant rollback are critical.

5) **Q:** Can blue/green work with microservices?
   - **A:** Yes, but it's complex. Each service needs blue/green environments, and you must coordinate cutover across services. Alternative: use canary deployments per service or feature flags to decouple deployment from release.

### Quick self-check (5 items)
- [ ] I can explain how blue/green deployment works (two environments, traffic switch).
- [ ] I can describe the cutover process and rollback mechanism.
- [ ] I can articulate the trade-off between cost and safety.
- [ ] I can explain how to handle database migrations in blue/green.
- [ ] I can compare blue/green to canary and rolling deployments.

## Links (NO duplication)
### Prerequisites
- [CI/CD basics](../quality/ci-cd-basics.md)
- [Reliability basics](reliability-basics.md)

### Related topics
- [Canary deployments](canary-deployments.md)
- [Rolling deployments](rolling-deployments.md)
- [Feature flags](feature-flags.md)

### Compare with
- [Canary deployments](canary-deployments.md) — Canary is gradual rollout; blue/green is instant switch.

## Notes / Inbox
- Add examples from real projects: deploying Heimdall with blue/green, Opportunity Actions cutover strategy.
- Consider adding section on blue/green with feature flags (decoupling deployment from release).
- Link to specific cloud implementations (AWS ECS blue/green, Kubernetes strategies) when created.
