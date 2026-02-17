---
id: production-incident
title: "Production Incident Response"
type: playbook
category: incident-response
difficulty: hard
importance: 100
tags: [incident, on-call, debugging, reliability, production]
created_at: 2026-02-12
updated_at: 2026-02-12
---

# Production Incident Response

## Situation
**When this applies:**
- Service degradation or outage in production
- Customer-impacting bug discovered
- Alert fires for SLA breach, error rate spike, latency increase
- On-call paged for critical issue

**Common manifestations:**
- "Database queries timing out"
- "500 errors spiking to 15%"
- "Payment processing stopped"
- "Search returning no results"
- "API p99 latency 10x normal"

## Why this matters
- Poor incident response amplifies damage (downtime, revenue loss, reputation)
- Speed of recovery directly impacts customer trust and SLA
- How you handle pressure reveals technical depth and leadership
- Critical interview topic: demonstrates debugging, communication, ownership under stress
- Incident response skill separates senior engineers from junior

## My Process (Step-by-step)

### 1. Stop the bleeding
- **What:** Reduce customer impact immediately (rollback, disable feature, route around failure)
- **Why:** Limiting damage is priority #1; understanding root cause comes later
- **How:** Rollback recent deploy, disable feature flag, scale up, failover to backup
- **Example:** "API errors spiked after deploy 30 min ago → rollback immediately → errors drop to baseline"

### 2. Quick triage
- **What:** Assess impact, scope, severity urgency
- **Why:** Determines response level (who to wake up, how fast to act)
- **How:** Check: how many users? revenue impact? data corruption? workaround available?
- **Example:**
  - Impact: 20% of users seeing errors
  - Scope: Checkout flow only
  - Severity: Critical (revenue-impacting)
  - Timeline: Started 15 minutes ago

### 3. Coordinate response
- **What:** Establish communication channel, roles, escalation path
- **Why:** Prevents chaos, duplicate work, miscommunication
- **How:** Create incident Slack channel, assign IC (incident commander), page relevant engineers, update status page
- **Example:** "#incident-2026-02-12-checkout" → IC: you → notify: payments team, PM, VP Eng if >1hr

### 4. Diagnose with evidence
- **What:** Follow signal chain: metrics → logs → traces
- **Why:** Random guessing wastes time; systematic diagnosis finds root cause
- **How:** Order hypotheses by likelihood; test most probable first
- **Example:**
  - Metrics: p99 timeout on `/checkout` endpoint
  - Logs: DB connection pool exhausted
  - Traces: new query scanning 10M rows (missing index)
  - Root cause: Deploy added unindexed query

### 5. Implement safe fix
- **What:** Minimum change to restore service with rollback plan
- **Why:** Complex fixes under pressure introduce new bugs
- **How:** Smallest possible change, tested in staging if possible, gradual rollout, monitor closely
- **Example:** "Add index on `users.email` → test locally → apply to replica → promote replica → monitor p99"

### 6. Operational recovery
- **What:** Clean up side effects (stuck jobs, corrupted data, queued retries)
- **Why:** Service may be "up" but data inconsistent or backlog building
- **How:** Check for: unprocessed messages, failed transactions, inconsistent state; replay or cleanup
- **Example:** "500 orders stuck in 'processing' state → identify via query → manually trigger completion webhooks → validate"

### 7. Clear all-clear and communicate
- **What:** Confirm metrics back to normal, notify stakeholders, close incident channel
- **Why:** Prevents premature closure and ensures everyone knows issue is resolved
- **How:** Check: error rate <1%, latency <p99 SLA, no alerts, customers not reporting issues
- **Example:** "p99 back to 80ms, error rate 0.2%, no new reports → post update: incident resolved → close channel"

### 8. Blameless postmortem
- **What:** Document timeline, root cause, preventive actions
- **Why:** Learn from failure; prevent recurrence
- **How:** Write: what happened (timeline), why (root cause), how to prevent (action items with owners/dates)
- **Example:**
  - **What:** Checkout down 45 min
  - **Why:** Unindexed query from deploy
  - **Prevent:** (1) Add query analysis to CI, (2) staging DB with prod-size data, (3) alert on slow query p95

## Success Signals
**You know it's working when:**
- [ ] Mean time to recovery (MTTR) <1 hour for critical incidents
- [ ] Customer impact minimized (complaints stop after mitigation)
- [ ] Team aligned and informed (no confusion or duplicate work)
- [ ] Root cause identified and documented
- [ ] Preventive action items completed within 2 weeks

## Common Pitfalls (and how to avoid them)

### Pitfall 1: Trying to debug before stopping the bleeding
- **Why it happens:** Curiosity, ego ("I can fix this"), adrenaline
- **Impact:** Incident duration 3x longer; customer pain continues
- **Instead do:** Rollback first, understand later

### Pitfall 2: No coordination (lone-wolf debugging)
- **Why it happens:** Urgency, lack of process
- **Impact:** Duplicate work, missed updates, stakeholder frustration
- **Instead do:** Create incident channel, assign IC, communicate status every 15 min

### Pitfall 3: Complex fix under pressure
- **Why it happens:** "Let's fix it properly while we're here"
- **Impact:** New bugs introduced; longer downtime
- **Instead do:** Minimum viable fix now; schedule proper fix later

### Pitfall 4: Skipping postmortem
- **Why it happens:** Firefighting fatigue, move on to next task
- **Impact:** Same incident recurs; no learning
- **Instead do:** Block 1 hour within 48 hours; make it blameless

## Real-world Example
**Context:** E-commerce checkout failing with 500 errors, Black Friday traffic.

**What happened:** Alerts fired at 2pm: error rate 20%, revenue dropping.

**Process applied:**
1. **Stop bleeding:** Rolled back deploy from 1:30pm → errors dropped to 2% (still elevated)
2. **Triage:** Critical severity, $10k/min revenue loss, 30% of users affected
3. **Coordinate:** Created #incident-black-friday-checkout, IC assigned, paged payments & infra teams
4. **Diagnose:**
   - Metrics: DB CPU 95%, slow queries
   - Logs: Deadlocks on `orders` table
   - Traces: New inventory-check query holding locks
   - Hypothesis: Query added in previous deploy (rolled back) but DB connections still saturated
5. **Fix:** Restarted DB connection pool → CPU dropped to 40%, errors to 0.5%
6. **Recovery:** 200 orders stuck in "pending" → manual script to retry → all completed
7. **All-clear:** Error rate normal, revenue recovered, updated status page
8. **Postmortem:** Wrote timeline, root cause (connection pool leak), action items:
   - Add connection pool monitoring
   - Load-test new queries with prod-size data
   - Circuit breaker for inventory check

**Outcome:** Total downtime 28 minutes, $280k revenue lost (vs potential $2M if not resolved), zero repeat incidents.

## Interview Tips
### How to talk about this
- **Framework to use:** "Stop bleeding → triage → diagnose → fix → recover → prevent"
- **Key points to emphasize:** Speed, systematic debugging, communication, ownership, learning
- **Language that works:** "contained impact," "systematic diagnosis," "minimal fix," "blameless postmortem"

### Sample STAR response outline
- **Situation:** "[Service] down with [impact: users, revenue, SLA breach]"
- **Task:** "On-call, responsible for restoring service ASAP and preventing recurrence"
- **Action:** "Rolled back to stop bleeding, triaged severity, coordinated [team], diagnosed via [metrics/logs/traces], implemented [minimal fix], recovered [data/backlog]"
- **Result:** "Service restored in [X minutes], [impact limited: revenue, users], postmortem with [N] preventive actions completed"

### Red flags to avoid
- Don't hero-worship ("I saved the company")
- Don't skip communication ("I was too busy to update")
- Don't blame ("the other team broke it")
- Don't skip prevention ("we fixed it, moved on")

## Self-assessment
- [ ] I can describe a specific production incident I resolved
- [ ] I can explain the systematic diagnosis process I used
- [ ] I can state the minimal fix vs long-term solution
- [ ] I can quantify the impact (downtime, users, revenue)
- [ ] I can list preventive actions taken

## Related Playbooks
- [Applying Deep Technical Knowledge](../technical/applying-deep-knowledge.md)
- [Owning a Mistake](../leadership/owning-mistake.md)
- [Delivering Under Pressure](../leadership/delivering-under-pressure.md)

## Related Topics (knowledge base)
- [Observability Basics](../../topics/operations/observability-basics.md)
- [Circuit Breaker](../../topics/operations/circuit-breaker.md)
- [Timeouts](../../topics/operations/timeouts.md)
- [Retries and Backoff](../../topics/operations/retries-and-backoff.md)
- [Incident Management Basics](../../topics/operations/incident-management-basics.md)

## Notes / Refinements
- Different incident severities need different process (SEV1 vs SEV3)
- Communication frequency: SEV1 every 15 min, SEV2 every hour
- Practice incident response in game days / chaos engineering
- Postmortems should have action items with owners and deadlines, not vague "improve X"
