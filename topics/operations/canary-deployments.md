---
id: canary-deployments
title: "Canary Deployments"
type: pattern
status: learning
importance: 80
difficulty: medium
tags: [deployment, reliability, progressive-rollout, risk-management]
canonical: true
aliases: [canary-release, progressive-deployment]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Canary Deployments

## TL;DR (BLUF)
- Canary deployment gradually routes increasing percentages of traffic to a new version (5% → 10% → 50% → 100%), validating at each step.
- Use for risk mitigation: catch issues with small blast radius before full rollout.
- Key trade-off: slower rollout (safety) vs. instant deployment (speed).

## Definition
**What it is:** A deployment strategy that gradually increases traffic to a new version while monitoring for issues, allowing rollback before full deployment if problems arise.  
**Key terms:** canary (new version with small traffic), baseline (current version), progressive rollout, blast radius, traffic splitting, automated rollback, health metrics.

## Why it matters
- **Risk mitigation:** Issues affect only small percentage of users (5% vs. 100%).
- **Early detection:** Catch bugs, performance issues, or business metric drops before full rollout.
- **Data-driven decisions:** Monitor real production traffic to validate changes.
- **Cost-effective:** No need for 2x infrastructure (unlike blue/green).
- **Interview relevance:** Standard pattern in system design for safe deployments.

## Scope & Non-goals
**In scope:**
- Canary deployment mechanics (traffic splitting, progressive rollout).
- Health metrics for validation (error rate, latency, business metrics).
- Automated rollback triggers.

**Out of scope / NOT solved by this:**
- Specific traffic splitting implementations (service mesh, load balancer—tools vary).
- Feature flags (separate but complementary pattern).
- Database migrations (separate challenge).

## Mental model / Intuition
Think of canary deployments as **testing a coal mine with a canary bird**:
- Miners sent a canary into the mine first (sensitive to toxic gas).
- If the canary died, they knew the mine was unsafe.
- In software: send 5% of traffic to the new version (the canary).
- If metrics degrade, rollback before exposing all users.

The "canary" is a small, representative sample that validates safety for everyone.

## Decision rules (When to use / When not to use)
### Use it when
- Risk mitigation is critical (customer-facing services, high traffic).
- You need to validate with real production traffic (not just tests).
- Infrastructure cost matters (can't afford 2x for blue/green).
- Changes are risky (major refactors, new features, performance changes).

### Avoid it when
- Rollout must be instant (canary is gradual; use blue/green instead).
- Traffic is too low to get meaningful signal (can't validate with 5% of 10 users/day).
- Infrastructure doesn't support traffic splitting (no load balancer/service mesh).

## How I would use it (practical)
- **Context:** Deploying a new version of a Go API service on Kubernetes with 10,000 req/min.
- **Steps:**
  1. **Deploy canary (5% traffic):**
     - Deploy new version (`api-v2`) with 1 pod.
     - Configure load balancer: 95% to `api-v1`, 5% to `api-v2`.
     - Monitor for 10 minutes: error rate, P95 latency, business metrics (conversions).
  2. **Increase to 10%:**
     - If metrics stable: scale `api-v2` to 2 pods, adjust traffic to 90/10.
     - Monitor for 10 minutes.
  3. **Increase to 50%:**
     - If stable: scale `api-v2` to 5 pods, adjust traffic to 50/50.
     - Monitor for 30 minutes (larger sample).
  4. **Full rollout (100%):**
     - If stable: scale `api-v2` to 10 pods, route 100% traffic.
     - Monitor for 1 hour; then tear down `api-v1`.
  5. **Automated rollback:**
     - If error rate >0.5% or P95 latency >500ms at any stage: auto-revert to 100% `api-v1`.
- **What success looks like:** Gradual rollout over 1 hour; zero user-facing incidents; auto-rollback if metrics degrade.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Low risk (small blast radius; catch issues early).
  - Cost-effective (no 2x infrastructure; canary runs alongside baseline).
  - Data-driven (validate with real production traffic).
  - Flexible (can pause, rollback, or accelerate rollout).
- **Cons / Risks:**
  - Slower rollout (gradual vs. instant).
  - Complexity (requires traffic splitting and monitoring).
  - Version skew (two versions running concurrently; API compatibility needed).
  - Low traffic invalidates signal (can't validate with 5% of 10 users).

### Alternatives
- **[Blue/green deployments](blue-green-deployments.md):** Instant 100% cutover (faster but riskier).
- **Rolling deployments:** Replace instances one-by-one (simpler but slower rollback).
- **Feature flags:** Deploy code but keep features off; enable gradually (decouples deploy from release).

### How to choose
- **Risk-sensitive + cost-effective:** Canary.
- **Zero downtime + instant rollback:** Blue/green.
- **Simple services, minimal risk:** Rolling.
- **Gradual feature enablement:** Feature flags + canary.

## Failure modes & Pitfalls
- **Insufficient monitoring:** Deploy canary without tracking metrics; miss issues until full rollout.
- **Too-fast rollout:** Increase traffic too quickly (e.g., 5% → 100%); miss issues that appear at scale.
- **No automated rollback:** Manual intervention required; slow response to issues.
- **Version incompatibility:** Canary and baseline can't coexist (API breaking changes, shared state issues).
- **Low traffic validates poorly:** 5% of 100 users/day = 5 users; not enough to detect issues.

## Observability (How to detect issues)
- **Metrics (compare canary to baseline):**
  - Error rate (% of requests failing; flag if canary >baseline).
  - Latency (P95, P99; flag if canary significantly slower).
  - Business metrics (conversion rate, checkout success; flag drops).
  - Resource utilization (CPU, memory; flag if canary uses more).
- **Logs:** Compare error logs between canary and baseline (look for new errors).
- **Traces:** Validate canary handles requests correctly (no timeout spikes, correct logic).
- **Alerts:** Automated rollback triggers (error rate >0.5%, latency >500ms, etc.).

## Implementation notes
- **Checklist**
  - [ ] Define health metrics (error rate, latency, business metrics).
  - [ ] Set rollback thresholds (e.g., error rate >0.5%, latency >500ms).
  - [ ] Deploy canary (5% traffic) and monitor for 10 minutes.
  - [ ] Gradually increase traffic (10%, 25%, 50%, 100%) with validation at each step.
  - [ ] Automate rollback (if metrics degrade, revert to 100% baseline).
  - [ ] Keep baseline running until canary reaches 100% and stabilizes.
  - [ ] Tear down baseline after validation period.

- **Security / Compliance notes**
  - Ensure canary and baseline have identical security posture.
  - Audit rollout decisions (who approved, what metrics triggered rollback).

- **Performance notes**
  - Warm up canary instances (pre-load caches, connections).
  - Monitor resource usage (canary may have different performance characteristics).

- **Operational notes**
  - Document rollout plan (traffic percentages, durations, rollback triggers).
  - Runbook for manual rollback (if automation fails).

## Mini example (Kubernetes + Istio)
```yaml
# Baseline deployment (v1)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-v1
spec:
  replicas: 10
  selector:
    matchLabels:
      app: api
      version: v1
  template:
    metadata:
      labels:
        app: api
        version: v1
    spec:
      containers:
      - name: api
        image: myapp:v1.0

---
# Canary deployment (v2)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-v2
spec:
  replicas: 1  # Start small
  selector:
    matchLabels:
      app: api
      version: v2
  template:
    metadata:
      labels:
        app: api
        version: v2
    spec:
      containers:
      - name: api
        image: myapp:v2.0

---
# Istio VirtualService (traffic splitting)
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: api
spec:
  hosts:
  - api.example.com
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: api
        subset: v1
      weight: 95  # Baseline gets 95%
    - destination:
        host: api
        subset: v2
      weight: 5   # Canary gets 5%
```

## Common anti-patterns
- **Anti-pattern:** No monitoring during canary rollout.
  - **Why it's bad:** Issues go undetected until full rollout; defeats purpose of canary.
  - **Better approach:** Define metrics, monitor continuously, automate rollback.

- **Anti-pattern:** Jumping from 5% to 100% without intermediate steps.
  - **Why it's bad:** Issues that appear at scale (10% or 50% traffic) are missed.
  - **Better approach:** Gradual rollout (5% → 10% → 25% → 50% → 100%) with validation at each step.

- **Anti-pattern:** Manual rollback only.
  - **Why it's bad:** Slow response time; issues affect more users while waiting for human intervention.
  - **Better approach:** Automated rollback triggers (error rate, latency thresholds).

- **Anti-pattern:** Canary deployment with breaking API changes.
  - **Why it's bad:** Canary and baseline can't coexist (clients fail on one or the other).
  - **Better approach:** Ensure API compatibility; use versioned endpoints if breaking changes needed.

## Interview readiness
### "Explain it like I'm teaching"
Canary deployment is a gradual rollout strategy where you start by sending a small percentage of traffic (e.g., 5%) to a new version while keeping the rest on the current version. You monitor key metrics—error rate, latency, business metrics—and only increase traffic if the canary is healthy. If metrics degrade, you automatically rollback to the baseline. The process continues incrementally (5% → 10% → 50% → 100%) until full rollout. The main benefit is risk mitigation: issues affect only a small percentage of users, giving you time to catch and fix them before everyone is impacted. It's cost-effective (no 2x infrastructure like blue/green) but slower.

### Trap questions (with answers)
1) **Q:** How do you decide what percentage of traffic to route to the canary?
   - **A:** Start small (5% or less) to limit blast radius. Increase gradually based on traffic volume and confidence. High-traffic services can validate quickly with 5%; low-traffic services may need 10-25% for meaningful signal. Increase when metrics are stable (error rate, latency, business metrics within thresholds).

2) **Q:** What metrics do you monitor during a canary deployment?
   - **A:** 
     - **Technical:** Error rate (5xx, 4xx), latency (P95, P99), throughput.
     - **Resource:** CPU, memory, connection pool usage.
     - **Business:** Conversion rate, checkout success, feature usage.
     - Compare canary to baseline; flag if canary is significantly worse.

3) **Q:** How is canary different from A/B testing?
   - **A:** 
     - **Canary:** Deployment safety mechanism. Goal is to validate new version works correctly (no errors, latency OK). Success means 100% rollout.
     - **A/B testing:** Product experiment. Goal is to measure business impact (does variant B increase conversions?). Success means learning, not necessarily rollout.
     - Both use traffic splitting, but different goals.

4) **Q:** What if the canary looks healthy at 5% but breaks at 50%?
   - **A:** This validates the canary strategy. Issues that appear at scale (database load, connection pool exhaustion, memory leaks) are caught before full rollout. Rollback to baseline, investigate, fix, and retry. The gradual rollout prevents 100% of users from experiencing the issue.

5) **Q:** Can you combine canary with blue/green deployments?
   - **A:** Yes. Deploy green environment, then canary traffic from blue to green (5% → 100%). Once validated, fully cutover to green. This combines blue/green's instant rollback with canary's gradual validation. More complex but very safe.

### Quick self-check (5 items)
- [ ] I can explain how canary deployment works (gradual traffic increase).
- [ ] I can name at least 3 metrics to monitor during canary rollout.
- [ ] I can describe automated rollback triggers and why they're important.
- [ ] I can compare canary to blue/green (trade-offs, when to use each).
- [ ] I can explain how to handle version incompatibility in canary deployments.

## Links (NO duplication)
### Prerequisites
- [CI/CD basics](../quality/ci-cd-basics.md)
- [Reliability basics](reliability-basics.md)
- [Observability basics](observability-basics.md)

### Related topics
- [Blue/green deployments](blue-green-deployments.md)
- [Feature flags](feature-flags.md)
- [Rolling deployments](rolling-deployments.md)

### Compare with
- [Blue/green deployments](blue-green-deployments.md) — Blue/green is instant switch; canary is gradual rollout.

## Notes / Inbox
- Add examples from real projects: canary deployment of Heimdall changes, Opportunity Actions rollout strategy.
- Consider adding section on canary analysis tools (Kayenta, Flagger).
- Link to service mesh implementations (Istio, Linkerd) when created.
