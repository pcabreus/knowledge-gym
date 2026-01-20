---
id: rolling-deployments
title: "Rolling Deployments"
type: pattern
status: learning
importance: 75
difficulty: easy
tags: [deployment, reliability, zero-downtime, operations]
canonical: true
aliases: [rolling-update, incremental-deployment]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Rolling Deployments

## TL;DR (BLUF)
- Rolling deployment replaces instances of the old version with the new version incrementally (one-by-one or in batches).
- Use for zero-downtime deployments with minimal infrastructure overhead.
- Key trade-off: simplicity and low cost vs. slower rollback (compared to blue/green).

## Definition
**What it is:** A deployment strategy that gradually replaces running instances of the old version with new version instances, one at a time or in small batches, until all instances are updated.  
**Key terms:** instance replacement, batch size, health checks, rollback window, zero-downtime deployment, graceful shutdown, surge (extra instances during rollout).

## Why it matters
- **Zero downtime:** Users experience no service interruption during deployments.
- **Cost-effective:** No need for 2x infrastructure (unlike blue/green).
- **Simple to implement:** Standard strategy in Kubernetes, AWS ECS, and most orchestration platforms.
- **Lower risk than big-bang:** Gradual rollout limits blast radius (but slower than canary).
- **Interview relevance:** Common deployment pattern in system design discussions.

## Scope & Non-goals
**In scope:**
- Rolling deployment mechanics (instance replacement, batch sizing).
- When to use rolling vs. other strategies.
- Health check validation during rollout.

**Out of scope / NOT solved by this:**
- Database migrations (separate challenge; requires forward-compatible schemas).
- Specific platform implementations (Kubernetes, ECS—though concepts apply).
- Advanced progressive delivery (use canary for that).

## Mental model / Intuition
Think of rolling deployment as **replacing light bulbs in a chandelier one-by-one**:
- You don't turn off all lights at once (that's big-bang deployment).
- You replace one bulb at a time while the chandelier stays lit (zero downtime).
- If a new bulb is defective, you notice it before replacing all bulbs (limited blast radius).
- Eventually all bulbs are new (full rollout complete).

## Decision rules (When to use / When not to use)
### Use it when
- Zero downtime is required (customer-facing services).
- Infrastructure cost matters (can't afford 2x for blue/green).
- Deployment is low-risk (mature service, well-tested changes).
- Platform supports rolling updates natively (Kubernetes, ECS, etc.).

### Avoid it when
- Instant rollback is critical (use blue/green instead).
- Gradual validation needed (use canary instead).
- Version incompatibility (old and new can't coexist).
- Database migrations are complex (rolling + schema changes is tricky).

## How I would use it (practical)
- **Context:** Deploying a Go API service on Kubernetes with 10 replicas.
- **Steps:**
  1. **Configure rolling update:**
     - Max surge: 2 (can add up to 2 extra instances during rollout).
     - Max unavailable: 1 (at most 1 instance can be down during rollout).
     - This ensures 9-12 instances running at all times (zero downtime).
  2. **Deploy new version:**
     - Kubernetes creates 1 new pod (v2).
     - Waits for health check to pass (readiness probe).
     - If healthy, terminates 1 old pod (v1).
     - Repeats until all 10 pods are v2.
  3. **Monitor during rollout:**
     - Watch error rate, latency, health checks.
     - If issues detected, pause or rollback (but slower than blue/green).
  4. **Complete rollout:**
     - All 10 pods now running v2.
     - Total rollout time: ~5-10 minutes (depending on pod startup time).
- **What success looks like:** Zero downtime; gradual replacement over 10 minutes; health checks validate each instance.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Zero downtime (instances replaced incrementally).
  - Cost-effective (no 2x infrastructure).
  - Simple (native support in most platforms).
  - Lower blast radius than big-bang (issues affect only some instances).
- **Cons / Risks:**
  - Slower rollback (must redeploy old version, then roll back incrementally).
  - Version skew (old and new run concurrently; API compatibility needed).
  - No instant traffic switch (unlike blue/green).
  - Slower than canary for risk detection (all instances eventually replaced).

### Alternatives
- **[Blue/green deployments](blue-green-deployments.md):** Instant cutover, instant rollback (higher cost).
- **[Canary deployments](canary-deployments.md):** Gradual validation with traffic percentage (more complex).
- **Recreate deployment:** Stop all old, start all new (simple but has downtime).

### How to choose
- **Zero downtime + low cost:** Rolling.
- **Zero downtime + instant rollback:** Blue/green.
- **Zero downtime + gradual validation:** Canary.
- **Simple service, downtime acceptable:** Recreate.

## Failure modes & Pitfalls
- **Failed health checks:** New instance fails readiness probe; rollout gets stuck (must fix or rollback).
- **Too-fast rollout:** Batch size too large (e.g., replace 5 instances at once); issues affect many users.
- **Version incompatibility:** Old and new instances can't coexist (API breaking changes, shared state issues).
- **Slow startup:** New instances take 5+ minutes to start; rollout takes hours.
- **No monitoring:** Deploy without watching metrics; issues go undetected until all instances are new.

## Observability (How to detect issues)
- **Metrics:**
  - Deployment progress (% of instances updated).
  - Error rate (compare new instances to old; flag spikes).
  - Latency (P95/P99; flag if new instances slower).
  - Health check success rate (should be 100%).
- **Logs:** Compare logs between old and new instances (look for new errors).
- **Traces:** Validate new instances handle requests correctly.
- **Alerts:** Health check failures, error rate spike, rollout stalled.

## Implementation notes
- **Checklist**
  - [ ] Configure rollout strategy (max surge, max unavailable, batch size).
  - [ ] Define health checks (readiness and liveness probes).
  - [ ] Set reasonable timeout (pod startup + readiness check; e.g., 5 minutes).
  - [ ] Monitor during rollout (error rate, latency, health checks).
  - [ ] Test rollback procedure (redeploy old version if needed).
  - [ ] Use graceful shutdown (drain connections before terminating old instances).

- **Security / Compliance notes**
  - Ensure new instances have identical security posture (configs, secrets, IAM roles).
  - Audit rollout events (who deployed, when, what version).

- **Performance notes**
  - Warm up new instances (pre-load caches, connections) for faster readiness.
  - Use connection draining on old instances (graceful shutdown).

- **Operational notes**
  - Document rollback procedure (must be executable under pressure).
  - Monitor closely during first few instance replacements (catch issues early).

## Mini example (Kubernetes)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # Can add up to 2 extra pods during rollout
      maxUnavailable: 1  # At most 1 pod can be down during rollout
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: myapp:v2.0  # New version
        ports:
        - containerPort: 8080
        readinessProbe:    # Health check before routing traffic
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:     # Health check during runtime
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10

# Rollout process:
# 1. kubectl apply -f deployment.yaml
# 2. kubectl rollout status deployment/api
# 3. kubectl rollout undo deployment/api  (if rollback needed)
```

## Common anti-patterns
- **Anti-pattern:** No health checks defined.
  - **Why it's bad:** Broken instances get traffic; users see errors.
  - **Better approach:** Define readiness and liveness probes; only route traffic to healthy instances.

- **Anti-pattern:** Max unavailable = replicas (replace all at once).
  - **Why it's bad:** Defeats purpose of rolling deployment; equivalent to recreate (downtime).
  - **Better approach:** Keep max unavailable small (1-2) to ensure service always available.

- **Anti-pattern:** Version incompatibility (API breaking changes).
  - **Why it's bad:** Old and new instances can't coexist; clients fail on one or the other.
  - **Better approach:** Ensure backward compatibility; use versioned endpoints if breaking changes needed.

- **Anti-pattern:** No rollback plan.
  - **Why it's bad:** Issues discovered mid-rollout; no quick way to revert.
  - **Better approach:** Document rollback procedure (`kubectl rollout undo` or redeploy old version).

## Interview readiness
### "Explain it like I'm teaching"
Rolling deployment replaces old instances with new instances gradually—one at a time or in small batches. For example, if you have 10 instances, Kubernetes might start 1 new instance, wait for it to be healthy, then terminate 1 old instance. It repeats this until all 10 are the new version. The benefits are zero downtime (always have healthy instances serving traffic) and low cost (no 2x infrastructure like blue/green). The downsides are slower rollback (must redeploy old version and roll it out incrementally) and version skew during rollout (old and new coexist temporarily). It's the default strategy in most orchestration platforms because it's simple and effective.

### Trap questions (with answers)
1) **Q:** How is rolling deployment different from blue/green?
   - **A:** 
     - **Rolling:** Replaces instances incrementally (one-by-one); no 2x infrastructure; slower rollback.
     - **Blue/green:** Two full environments; instant traffic switch; instant rollback; 2x infrastructure cost.
     - Rolling is cost-effective; blue/green is faster and safer.

2) **Q:** What happens if a new instance fails health checks during rolling deployment?
   - **A:** The rollout pauses. Kubernetes (or orchestrator) won't terminate old instances if new instances aren't ready. You must fix the issue (code bug, config error) and redeploy, or rollback to the old version. This prevents broken instances from getting traffic.

3) **Q:** Can you have zero downtime with rolling deployments?
   - **A:** Yes, if configured correctly. Use `maxSurge` to allow extra instances during rollout and `maxUnavailable` to limit how many can be down at once. For example, with 10 replicas, maxSurge=2, maxUnavailable=1, you always have 9-12 healthy instances serving traffic (zero downtime).

4) **Q:** How do you handle database migrations with rolling deployments?
   - **A:** Use forward-compatible schemas (old and new code can both read/write). Run migrations before deploying new code. For complex changes, use expand/contract pattern: expand schema (add new fields), deploy new code (uses new fields), contract schema (remove old fields) after all instances are new.

5) **Q:** What's the difference between rolling and canary deployments?
   - **A:** 
     - **Rolling:** Replaces all instances incrementally; no traffic control (instances get equal traffic).
     - **Canary:** Routes small percentage of traffic to new version; gradually increases (5% → 100%); monitors metrics before full rollout.
     - Rolling is simpler; canary provides better risk mitigation and validation.

### Quick self-check (5 items)
- [ ] I can explain how rolling deployment works (incremental instance replacement).
- [ ] I can describe the role of health checks in rolling deployments.
- [ ] I can articulate the trade-off between cost and rollback speed.
- [ ] I can compare rolling to blue/green and canary deployments.
- [ ] I can explain how to configure zero-downtime rolling updates (maxSurge, maxUnavailable).

## Links (NO duplication)
### Prerequisites
- [CI/CD basics](../quality/ci-cd-basics.md)
- [Reliability basics](reliability-basics.md)

### Related topics
- [Blue/green deployments](blue-green-deployments.md)
- [Canary deployments](canary-deployments.md)
- [Feature flags](feature-flags.md)

### Compare with
- [Blue/green deployments](blue-green-deployments.md) — Blue/green is instant switch; rolling is incremental replacement.
- [Canary deployments](canary-deployments.md) — Canary controls traffic percentage; rolling replaces instances.

## Notes / Inbox
- Add examples from real projects: rolling updates for Heimdall, Opportunity Actions deployment strategy.
- Consider adding section on rollout velocity (how to speed up or slow down rollouts).
- Link to Kubernetes rolling update details when created.
