---
id: debugging-performance-issues
title: "Debugging Performance Issues"
type: playbook
category: technical
difficulty: hard
importance: 95
tags: [performance, debugging, profiling, optimization, observability]
created_at: 2026-02-12
updated_at: 2026-02-12
---

# Debugging Performance Issues

## Situation
**When this applies:**
- System slower than expected or degrading over time
- User complaints about latency
- Metrics show p95/p99 spikes
- Resource utilization (CPU, memory, disk) unexpectedly high

**Common manifestations:**
- "API response time increased from 100ms to 2s"
- "Database queries timing out"
- "Frontend feels laggy"
- "Background jobs backing up"
- "Memory usage growing unbounded"

## Why this matters
- Performance issues directly impact user experience and revenue
- Systematic debugging finds root cause faster than guessing
- Performance problems compound: slow query → timeout → retry storm → outage
- Critical interview topic: demonstrates technical depth, systematic thinking, production experience
- Senior engineers diagnose and fix, not just escalate

## My Process (Step-by-step)

### 1. Establish baseline and quantify regression
- **What:** Define what "slow" means with numbers
- **Why:** "Feels slow" is subjective; numbers are objective
- **How:** Compare current vs historical metrics (p50, p95, p99, max)
- **Example:**
  - **Current:** p95 = 2.5s, p99 = 5s
  - **Baseline (last week):** p95 = 200ms, p99 = 500ms
  - **Regression:** 10x slower at p95, started 3 days ago

### 2. Correlate with recent changes
- **What:** Identify what changed when performance degraded
- **Why:** Most regressions trace to recent deploys, config changes, or traffic patterns
- **How:** Check: deploys, config changes, DB migrations, traffic spikes, dependency updates
- **Example:**
  - **Timeline:** Performance degraded starting Tuesday 2pm
  - **Changes:** Deploy at Monday 5pm added new query with `JOIN` on 10M row table
  - **Hypothesis:** New query causing slow DB performance

### 3. Follow observability chain: metrics → logs → traces
- **What:** Systematic narrowing from high-level to specific
- **Why:** Random log grepping wastes time; metrics point to where to look
- **How:** 
  - **Metrics:** Which endpoint/operation is slow?
  - **Logs:** What errors or warnings correlate?
  - **Traces:** Where in the request path is time spent?
- **Example:**
  - **Metrics:** `/api/orders` endpoint p95 = 3s (other endpoints fine)
  - **Logs:** "Query took 2.8s" repeating for `SELECT * FROM orders WHERE user_id = ?`
  - **Trace:** 95% of time in DB query, 5% in application code
  - **Root cause:** DB query is bottleneck

### 4. Reproduce locally or in staging
- **What:** Isolate issue in controlled environment
- **Why:** Production debugging is risky; staging allows experimentation
- **How:** Use production-like data size, run query/request with same parameters
- **Example:**
  - **Staging:** Loaded 10M orders (prod-size dataset)
  - **Reproduced:** Same query takes 2.5s in staging
  - **Next:** Can now profile and test fixes safely

### 5. Profile to find hotspot
- **What:** Use profiling tools to identify expensive operations
- **Why:** Intuition misleads; profiler shows truth
- **How:** 
  - **DB:** `EXPLAIN ANALYZE` for query plan
  - **Backend:** pprof (Go), cProfiler (Python), perf (Linux)
  - **Frontend:** Chrome DevTools, Lighthouse
- **Example:**
  - **DB:** `EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = ?`
  - **Result:** Seq scan on 10M rows (missing index on `user_id`)
  - **Hotspot:** Index lookup would reduce from O(n) to O(log n)

### 6. Hypothesize and test fix
- **What:** Form hypothesis about root cause, test with smallest change
- **Why:** Complex fixes under pressure introduce bugs
- **How:** Make minimal change, measure before/after, rollback if worse
- **Example:**
  - **Hypothesis:** Missing index on `orders.user_id` causes seq scan
  - **Fix:** `CREATE INDEX CONCURRENTLY ON orders(user_id)`
  - **Test in staging:** Query drops from 2.5s to 50ms (50x faster)
  - **Deploy to prod:** p95 drops to 180ms (validates hypothesis)

### 7. Monitor for regression
- **What:** Track metric after fix to ensure it holds
- **Why:** Fixes can degrade over time (data growth, cache expiry)
- **How:** Set alerts on the metric that was broken
- **Example:**
  - **Alert:** `/api/orders` p95 >500ms for 10 minutes
  - **Dashboard:** Graph showing before/after fix with annotation
  - **Review:** Check metric weekly for 1 month

### 8. Document root cause and prevention
- **What:** Write postmortem or runbook entry
- **Why:** Prevents recurrence and helps others learn
- **How:** Document: symptom, root cause, fix, prevention (tests, alerts, process)
- **Example:**
  ```markdown
  ## Incident: API latency spike
  
  **Symptom:** /api/orders p95 2.5s (baseline: 200ms)
  
  **Root cause:** Added query with JOIN on unindexed column (user_id)
  
  **Fix:** Created index on orders.user_id → p95 dropped to 180ms
  
  **Prevention:**
  - CI check: EXPLAIN ANALYZE on new queries in migration tests
  - Staging DB with prod-size data (10M rows)
  - Alert on p95 >500ms
  ```

## Success Signals
**You know it's working when:**
- [ ] Performance metric returns to baseline or better
- [ ] Root cause identified with evidence (not guessing)
- [ ] Fix validated in staging before production
- [ ] Alert configured to catch future regressions
- [ ] Team references your debugging process for similar issues

## Common Pitfalls (and how to avoid them)

### Pitfall 1: Optimizing without measuring
- **Why it happens:** Intuition, premature optimization
- **Impact:** Waste time optimizing non-bottleneck; miss real issue
- **Instead do:** Profile first, optimize hotspot only

### Pitfall 2: Testing fix in production first
- **Why it happens:** Urgency, no staging environment
- **Impact:** Risk making it worse, creating new incident
- **Instead do:** Reproduce in staging, validate fix there first

### Pitfall 3: Complex fix under pressure
- **Why it happens:** "Let's rewrite this properly"
- **Impact:** Introduce bugs, extend incident
- **Instead do:** Minimal fix to restore performance, rewrite later if needed

### Pitfall 4: Not setting up prevention
- **Why it happens:** Fire-and-forget, moving to next task
- **Impact:** Same issue recurs in weeks/months
- **Instead do:** Add test, alert, or process change immediately

## Real-world Example
**Context:** Payment processing p99 spiked from 500ms to 8s; users complaining about slow checkout.

**What happened:** Started Friday morning; impacting revenue during peak traffic.

**Process applied:**
1. **Baseline:** Current p99 = 8s, baseline = 500ms (16x regression), started Thursday 8pm
2. **Correlated changes:**
   - **Deploy Thursday 6pm:** Added fraud detection call to external API
   - **Hypothesis:** External API timeout causing latency
3. **Observability chain:**
   - **Metrics:** `/api/checkout` slow, other endpoints fine
   - **Logs:** "Fraud API timeout after 5s" warnings
   - **Traces:** 5-6s spent waiting for fraud API (network timeout)
4. **Reproduced in staging:** Same timeout; fraud API p99 = 5s (way above our 500ms SLA)
5. **Profiled:**
   - Traced requests to fraud API
   - Found: synchronous call blocking checkout response
   - Hotspot: waiting for I/O, not compute
6. **Hypothesis + fix:**
   - **Hypothesis:** Sync fraud call blocks checkout; API is slow/unreliable
   - **Fix (minimal):** Reduce fraud API timeout from 5s to 1s; queue manual review if timeout
   - **Test in staging:** p99 drops to 1.2s (better, not perfect)
   - **Deploy to prod with feature flag:** p99 drops to 1.3s
7. **Monitored:** Alert on p99 >2s; dashboarded fraud API timeout rate (15% timing out)
8. **Documented + longer-term fix:**
   - **Root cause:** Synchronous fraud check, unreliable third-party API
   - **Immediate fix:** Timeout at 1s, queue for review
   - **Long-term (next sprint):** Async fraud check (queue-based), don't block checkout
   - **Prevention:** SLA requirement for all third-party APIs, circuit breaker pattern

**Outcome:** p99 restored to 1.3s in 2 hours, revenue impact minimized, async fix shipped next week (p99 = 400ms).

## Interview Tips
### How to talk about this
- **Framework to use:** "Baseline → correlate → metrics→logs→traces → reproduce → profile → fix → monitor → prevent"
- **Key points to emphasize:** Systematic approach, evidence-based, minimal fix, prevention
- **Language that works:** "established baseline," "correlated with," "profiled and found," "validated in staging," "monitored for regression"

### Sample STAR response outline
- **Situation:** "[System/endpoint] experienced [N]x performance regression affecting [users/revenue]"
- **Task:** "Needed to identify root cause and restore performance quickly"
- **Action:** "Established baseline (was X, now Y), correlated with recent changes, followed metrics→logs→traces, reproduced in staging, profiled to find hotspot [Z], implemented minimal fix, validated, deployed with monitoring"
- **Result:** "Restored performance to [metric], root cause [identified], prevented recurrence with [test/alert/process]"

### Red flags to avoid
- Don't claim you "just knew" (must show systematic process)
- Don't skip validation ("I deployed to prod and hoped")
- Don't forget prevention (same issue next month is negligence)
- Don't skip metrics (must quantify before/after)

## Self-assessment
- [ ] I can describe specific performance issue I debugged
- [ ] I can explain the systematic process I followed
- [ ] I can state the profiling tool I used and what it showed
- [ ] I can quantify the improvement (Nx faster, p95/p99 numbers)
- [ ] I can describe the prevention I implemented

## Related Playbooks
- [Production Incident Response](../incident-response/production-incident.md)
- [Applying Deep Technical Knowledge](applying-deep-knowledge.md)
- [Evaluating Project Success](../leadership/evaluating-project-success.md)

## Related Topics (knowledge base)
- [Observability Basics](../../topics/operations/observability-basics.md)
- [Performance Basics](../../topics/system-design/performance-basics.md)
- [Slow Query Diagnosis](../../topics/databases/slow-query-diagnosis.md)
- [PostgreSQL Query Planning](../../topics/databases/query-planning.md)
- [EXPLAIN](../../topics/databases/explain.md)
- [Caching Strategy](../../topics/system-design/caching-strategy.md)
- [Backpressure](../../topics/system-design/backpressure.md)

## Notes / Refinements
- Most performance issues are data problems (missing index, N+1 queries, inefficient query)
- Second most common: external API latency (use timeouts, circuit breakers, async)
- Profile before optimizing: intuition wrong 80% of the time
- Staging must have prod-size data to catch performance issues
- Tools by language: Go (pprof), Python (cProfile), Java (JProfiler, async-profiler), JS (Chrome DevTools)
