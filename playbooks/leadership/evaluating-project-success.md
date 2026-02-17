---
id: evaluating-project-success
title: "Evaluating Project Success"
type: playbook
category: leadership
difficulty: medium
importance: 85
tags: [metrics, success-criteria, measurement, product-engineering-alignment, observability]
created_at: 2026-02-12
updated_at: 2026-02-12
---

# Evaluating Project Success

## Situation
**When this applies:**
- Starting new feature or project
- Post-launch review of shipped work
- Stakeholder asks "Did this work?"
- Need to decide: iterate, optimize, or pivot

**Common manifestations:**
- "We shipped feature X, now what?"
- "How do we know if this migration was successful?"
- "Product wants to know if users are adopting the new flow"
- "Should we invest more in this or move on?"

## Why this matters
- Shipping without measuring success is guessing, not engineering
- Misaligned success criteria cause friction between product and engineering
- Clear metrics enable data-driven iteration vs opinion-driven debates
- Critical interview topic: demonstrates business acumen, measurement discipline, outcome focus
- Senior engineers define success, not just implement requirements

## My Process (Step-by-step)

### 1. Align with product on success criteria before implementation
- **What:** Define what "good" looks like across business and technical dimensions
- **Why:** Post-launch measurement fails without pre-launch agreement
- **How:** Schedule alignment meeting; ask: "How will we know this worked?"
- **Example:**
  - **Bad:** "We'll see if users like it"
  - **Good:** "Success = 30% adoption in 4 weeks, <500ms p95 latency, <1% error rate"

### 2. Define business metrics
- **What:** Quantify user behavior and business outcomes
- **Why:** Engineering exists to drive business value; must measure it
- **How:** Common metrics: adoption rate, conversion, usage frequency, retention, revenue
- **Example:**
  - **Feature:** New checkout flow
  - **Business metrics:**
    - Adoption: 50% of users try new flow within 2 weeks
    - Conversion: Cart-to-purchase rate >15% (current: 12%)
    - Retention: 7-day return rate >40%

### 3. Define technical metrics
- **What:** Quantify system health and performance
- **Why:** Business success built on technical reliability
- **How:** Use RED (Rate, Errors, Duration) or USE (Utilization, Saturation, Errors) framework
- **Example:**
  - **Rate:** 1000 TPS (current: 800 TPS)
  - **Errors:** <1% error rate (current: 0.5%)
  - **Duration:** p95 <500ms, p99 <1s (current: p95 400ms, p99 800ms)
  - **Resource:** CPU <60%, memory <70%

### 4. Document metrics in technical proposal
- **What:** Write down success criteria in design doc or RFC
- **Why:** Creates shared understanding and prevents "moving goalposts"
- **How:** Add "Success Criteria" section to design doc with specific numbers and timeframes
- **Example:**
  ```markdown
  ## Success Criteria
  
  **Business (measured at 4 weeks post-launch):**
  - 50% adoption (users trying new flow)
  - 15% conversion (cart → purchase)
  - <5% support tickets related to checkout
  
  **Technical (monitored continuously):**
  - p95 latency <500ms
  - Error rate <1%
  - CPU utilization <60%
  - Zero data loss incidents
  
  **Go/No-Go decision at 2 weeks:**
  - If adoption <20%, roll back and iterate
  - If error rate >2%, investigate and fix before full rollout
  ```

### 5. Configure dashboards and alerts before rollout
- **What:** Set up monitoring infrastructure before shipping
- **Why:** Can't measure success without instrumentation
- **How:** Create dashboards (Grafana/DataDog), add custom metrics, configure alerts
- **Example:**
  - **Dashboard panels:**
    - Adoption rate (new flow users / total users)
    - Conversion funnel (cart → payment → confirm → success)
    - Latency histogram (p50, p95, p99)
    - Error rate by type
  - **Alerts:**
    - Error rate >1% for 5 minutes
    - p95 latency >500ms for 10 minutes
    - Conversion drops >10% from baseline

### 6. Review metrics in weekly syncs
- **What:** Regular review with product and engineering to assess progress
- **Why:** Continuous monitoring catches issues early and informs decisions
- **How:** Weekly 30-min meeting: review dashboard, discuss trends, decide actions
- **Example:**
  - **Week 1:** Adoption 15% (on track), conversion 13% (below target), p95 300ms (good)
  - **Week 2:** Adoption 35% (ahead), conversion 14% (improving), error spike investigated
  - **Week 3:** Adoption 52% (success!), conversion 16% (exceeded!), stable performance

### 7. Decide: iterate, optimize, or pivot
- **What:** Based on data, choose next step
- **Why:** Metrics inform strategy; don't iterate blindly
- **How:** Use decision framework based on success criteria
- **Example:**
  - **Iterate (meeting criteria):** "Conversion hit 16%, but p95 latency 450ms. Optimize for speed."
  - **Pivot (not meeting criteria):** "Adoption only 10% at week 4. Users prefer old flow. Roll back, gather feedback, redesign."
  - **Scale (exceeding):** "Conversion 18%, adoption 60%, stable. Remove old flow, invest in additional features."

### 8. Document learnings and close loop
- **What:** Write retrospective with what worked, what didn't, what we learned
- **Why:** Organizational learning; informs future projects
- **How:** Doc with: hypothesis, results, surprises, action items
- **Example:**
  ```markdown
  ## Retrospective: New Checkout Flow
  
  **Hypothesis:** Simplified 3-step checkout would improve conversion
  
  **Results:**
  - ✅ Conversion improved 12% → 16% (33% increase)
  - ✅ Adoption 52% in 4 weeks (exceeded 50% target)
  - ⚠️ Mobile conversion lower (14%) than desktop (18%)
  
  **Surprises:**
  - Users confused by payment method selector (10% support tickets)
  - International users had higher latency (800ms p95) due to address validation
  
  **Actions:**
  - Improve payment method UX (P0, next sprint)
  - Optimize international address validation (P1, Q2)
  - Pattern: Always A/B test checkout changes (too risky to guess)
  ```

## Success Signals
**You know it's working when:**
- [ ] Success criteria defined before implementation
- [ ] Product and engineering aligned on metrics
- [ ] Dashboards configured and alerts firing appropriately
- [ ] Data drives iteration decisions (not opinions)
- [ ] Team can articulate "why" behind next steps

## Common Pitfalls (and how to avoid them)

### Pitfall 1: Measuring too late (post-launch scramble)
- **Why it happens:** Urgency, assuming "we'll figure it out"
- **Impact:** Can't measure what you didn't instrument; guessing vs knowing
- **Instead do:** Define metrics and set up monitoring during planning phase

### Pitfall 2: Misaligned success criteria (product vs engineering)
- **Why it happens:** Siloed planning, different incentives
- **Impact:** Product frustrated by tech focus; engineering frustrated by feature churn
- **Instead do:** Joint definition of business + technical metrics

### Pitfall 3: Vanity metrics (measures activity, not outcomes)
- **Why it happens:** Easy to measure; feels good
- **Impact:** High adoption but low value; false sense of success
- **Instead do:** Measure outcomes: retention, conversion, revenue—not just impressions or clicks

### Pitfall 4: Not revisiting metrics (set and forget)
- **Why it happens:** Fire-and-forget culture, moving to next project
- **Impact:** Missed opportunities to optimize; degradation goes unnoticed
- **Instead do:** Weekly reviews for first month, monthly thereafter

## Real-world Example
**Context:** Migrated payment processing from synchronous to async (queued) to improve checkout latency.

**What happened:** Team shipped without clear success definition; debates about "did it work?"

**Process applied:**
1. **Aligned with product:** Success = faster checkout (p95 <500ms), no drop in conversion, <1% errors
2. **Business metrics:**
   - Conversion rate (target: ≥12%, current baseline)
   - Cart abandonment rate (target: ≤15%)
   - Support tickets about payment failures (target: <10/week)
3. **Technical metrics:**
   - Checkout p95 latency (target: <500ms, current: 1.2s)
   - Payment processing error rate (target: <1%)
   - Queue depth (target: <1000 messages)
   - Payment confirmation time (target: <10s)
4. **Documented in RFC:**
   ```markdown
   ## Success Criteria (4 weeks post-launch)
   - p95 latency <500ms (currently 1.2s) → 60% improvement
   - Conversion ≥12% (maintain or improve)
   - Error rate <1%
   - Support tickets <10/week (currently 15/week)
   ```
5. **Configured before rollout:**
   - Grafana dashboard with 8 panels (latency, errors, queue, conversion funnel)
   - Alerts: p95 >500ms, error rate >1%, queue depth >5000
6. **Weekly reviews:**
   - **Week 1:** p95 400ms (✅), conversion 12.5% (✅), 2 errors from timeout config (investigated)
   - **Week 2:** p95 350ms (excellent), conversion 13% (improving!), queue stable
   - **Week 4:** p95 380ms (sustained), conversion 13.5% (better than baseline), support tickets down to 6/week
7. **Decision:** Optimize further—95% of benefit achieved, next: reduce payment confirmation time from 8s to 5s
8. **Documented learning:** "Async improved latency 70% and conversion 12.5%. Surprise: queue rarely exceeded 500 messages (over-provisioned). Future: start smaller, scale based on data."

**Outcome:** Clearly successful migration, data-driven next steps, stakeholder confidence in team's delivery.

## Interview Tips
### How to talk about this
- **Framework to use:** "Align → define (business + technical) → document → instrument → review → decide → learn"
- **Key points to emphasize:** Pre-launch alignment, data-driven decisions, stakeholder collaboration, learning
- **Language that works:** "aligned on success criteria," "defined metrics," "configured dashboards before launch," "data showed," "decided to iterate based on"

### Sample STAR response outline
- **Situation:** "Launched [feature/project] with [business goal: improve conversion, reduce cost, scale]"
- **Task:** "Needed to evaluate success objectively and decide next steps"
- **Action:** "Aligned with product on success criteria (business + technical metrics), documented in design doc, configured dashboards/alerts before rollout, reviewed weekly, used data to decide iteration strategy"
- **Result:** "[Metric improved X%], [business outcome achieved], [decided to iterate/optimize/scale based on data], [stakeholder confidence improved]"

### Red flags to avoid
- Don't claim "everyone loved it" (subjective, not measurable)
- Don't skip pre-launch alignment ("we figured it out later")
- Don't measure only technical metrics (business outcomes matter)
- Don't forget the decision (measurement without action is pointless)

## Self-assessment
- [ ] I can describe how I defined success criteria before launching
- [ ] I can list both business and technical metrics I tracked
- [ ] I can explain how I instrumented for measurement (dashboards, alerts)
- [ ] I can state the data-driven decision I made (iterate/optimize/pivot)
- [ ] I can quantify the outcome and learning

## Related Playbooks
- [Making Trade-off Decisions](../decision-making/making-trade-offs.md)
- [Delivering Under Pressure](delivering-under-pressure.md)
- [Leadership Without Authority](leadership-without-authority.md)
- [Applying Deep Technical Knowledge](../technical/applying-deep-knowledge.md)

## Related Topics (knowledge base)
- [Observability Basics](../../topics/operations/observability-basics.md)
- [Service Level Indicator (SLI)](../../topics/operations/service-level-indicator.md)
- [Service Level Objective (SLO)](../../topics/operations/service-level-objective.md)
- [Performance Basics](../../topics/system-design/performance-basics.md)
- (TODO) [A/B Testing](../../topics/system-design/ab-testing.md)

## Notes / Refinements
- Balance leading indicators (early signals) vs lagging indicators (final outcomes)
- Common mistake: over-instrumenting (100 metrics) vs under-instrumenting (0 metrics)
- For large launches: define go/no-go criteria at checkpoints (e.g., week 1, week 2)
- Metrics should inform decisions, not dictate blindly—context matters
- Document success criteria even for "small" projects—builds metric discipline
