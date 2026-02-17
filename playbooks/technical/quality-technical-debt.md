---
id: quality-technical-debt
title: "Quality and Technical Debt"
type: playbook
category: technical
difficulty: hard
importance: 85
tags: [technical-debt, quality, refactoring, maintainability, velocity]
created_at: 2026-02-12
updated_at: 2026-02-12
---

# Quality and Technical Debt

## Situation
**When this applies:**
- Code quality degrading, slowing feature velocity
- Accumulating tech debt from rushed delivery
- Legacy system causing repeated incidents
- Team velocity declining despite adding engineers

**Common manifestations:**
- "Every feature takes 3x longer than it should"
- "We spend 50% of time fixing bugs, not shipping features"
- "This module hasn't been touched in 2 years because no one understands it"
- "On-call load increasing due to fragile systems"

## Why this matters
- Tech debt compounds like financial debt: small today, crippling tomorrow
- Poor quality causes incidents, slows delivery, burns out teams
- Strategic debt paydown unlocks velocity and reliability
- Critical interview topic: demonstrates judgment, long-term thinking, business acumen
- Senior engineers balance delivery with sustainability

## My Process (Step-by-step)

### 1. Classify debt
- **What:** Categorize by type and impact
- **Why:** Not all debt is equal; prioritize high-impact
- **How:** Categorize: reliability (incidents), scalability (growth blocker), velocity (slows features), security, maintainability
- **Example:**
  - **Reliability debt:** Auth service crashing weekly (incidents)
  - **Velocity debt:** Payment module 10k lines, no tests (slows features)
  - **Scalability debt:** DB single-node (growth blocker)

### 2. Quantify impact
- **What:** Measure cost in incidents, time, revenue, on-call load
- **Why:** Data persuades stakeholders better than "it's bad"
- **How:** Track: incident frequency, MTTR, feature lead time, on-call hours, customer complaints
- **Example:**
  - Auth debt: 5 incidents/month, 3 hours MTTR = 15 hours/month firefighting
  - Payment debt: Features in that module take 2x longer (50% velocity loss)
  - "Tech debt costs ~40 eng-hours/month in incidents + slowed delivery"

### 3. Prioritize debt that reduces risk or unlocks roadmap
- **What:** Pay down debt strategically, not randomly
- **Why:** Can't fix all debt; focus on high-ROI
- **How:** Prioritize: incident-causing, roadmap-blocking, high-traffic, security-critical
- **Example:**
  - **High priority:** Auth refactor (reduces incidents, unlocks SSO feature)
  - **Low priority:** Old admin UI code (low traffic, no incidents)

### 4. Pay down incrementally (boy-scout rule)
- **What:** Improve code as you touch it, don't wait for "refactor quarter"
- **Why:** Big rewrites fail; small improvements compound
- **How:** When fixing bug or adding feature: refactor surrounding code, add tests, improve naming
- **Example:** "Adding payment retry feature → refactored payment module into 3 smaller modules, added 10 tests, extracted config"

### 5. Add guardrails
- **What:** Prevent new debt with tests, linting, CI gates, observability
- **Why:** Easier to prevent than fix retroactively
- **How:** Enforce: test coverage thresholds, linting, pre-commit hooks, code review standards
- **Example:**
  - CI gate: >80% test coverage for new code
  - Linter: flag functions >50 lines, cyclomatic complexity >10
  - Alert on slow query p95 > 1s

### 6. Get buy-in using data
- **What:** Present debt paydown as investment with ROI
- **Why:** PMs prioritize features; data shows debt paydown is a feature
- **How:** Frame as: "Investing N weeks reduces incidents 50%, unlocks X feature"
- **Example:** "Proposal: 2 weeks to refactor auth → reduce incidents from 5/month to 1/month → save 10 eng-hours/month → ROI positive in 6 weeks"

### 7. Track like product work
- **What:** Create tickets, milestones, measure progress
- **Why:** "We'll fix it someday" never happens
- **How:** Add to backlog, timebox (e.g., 20% sprint capacity), track completion
- **Example:** "Sprint allocation: 60% features, 20% tech debt, 20% bugs. Track: debt tickets closed, incident rate trend."

## Success Signals
**You know it's working when:**
- [ ] Incident frequency declining
- [ ] Feature lead time improving
- [ ] On-call load decreasing
- [ ] Team velocity increasing (not decreasing)
- [ ] Code review feedback improving (cleaner code)

## Common Pitfalls (and how to avoid them)

### Pitfall 1: "We'll pay it down after we ship"
- **Why it happens:** Pressure, optimism
- **Impact:** Debt accumulates; "after" never comes
- **Instead do:** Pay incrementally; boy-scout rule

### Pitfall 2: Big-bang rewrites
- **Why it happens:** Frustration, "we'll do it right this time"
- **Impact:** 6-month rewrites fail or ship with new bugs
- **Instead do:** Incremental refactoring; ship small, ship often

### Pitfall 3: Not quantifying cost
- **Why it happens:** "It's obviously bad"
- **Impact:** Can't get buy-in; features always win
- **Instead do:** Measure incident hours, velocity impact, on-call load

### Pitfall 4: Treating all debt equally
- **Why it happens:** "We should fix everything"
- **Impact:** Spread thin, high-impact debt ignored
- **Instead do:** Prioritize: incident-causing, roadmap-blocking, high-traffic

## Real-world Example
**Context:** Payment module: 8k lines, no tests, 10 incidents/quarter, slowing all payment features.

**What happened:** PM wanted new payment provider feature; would take 6 weeks due to debt. Team wanted to "pause features and refactor."

**Process applied:**
1. **Classified:** Velocity debt (slows features) + reliability debt (incidents)
2. **Quantified:**
   - Incidents: 10/quarter, 2 hours MTTR each = 20 hours firefighting
   - Velocity: Payment features 3x slower than other modules
   - Cost: ~60 eng-hours/quarter lost
3. **Prioritized:** Payment module HIGH (roadmap-blocking for new provider feature)
4. **Incremental plan:**
   - **Week 1-2:** Extract Stripe logic into StripeProvider class (touch only Stripe paths)
   - **Week 3-4:** Add new BraintreeProvider using same interface
   - **Week 5-6:** Add integration tests, refactor error handling
   - NO big rewrite; refactor while shipping feature
5. **Guardrails:**
   - Required: >70% test coverage for payment module
   - CI: lint for functions >50 lines
   - Alert: payment error rate >1%
6. **Got buy-in:** "2 weeks refactor upfront saves 4 weeks on new provider feature → net 2 weeks faster + reduces future incidents"
7. **Tracked:** Sprint tickets for refactor steps, tracked incident trend

**Outcome:** Shipped new provider on time, incidents dropped from 10/quarter to 2/quarter, future payment features 2x faster.

## Interview Tips
### How to talk about this
- **Framework to use:** "Classify → quantify → prioritize → incremental → guardrails → buy-in → track"
- **Key points to emphasize:** Data-driven, business ROI, incremental, measurable improvement
- **Language that works:** "quantified impact," "prioritized high-ROI," "paid down incrementally," "resulted in"

### Sample STAR response outline
- **Situation:** "[System/module] had [tech debt type] causing [impact: incidents, slow velocity]"
- **Task:** "Needed to pay down debt without stopping feature delivery"
- **Action:** "Quantified cost [X hours/incidents], prioritized [high-impact debt], paid down incrementally while shipping [feature], added guardrails [tests/CI]"
- **Result:** "Incidents reduced [%], velocity improved [%], ROI positive in [timeframe]"

### Red flags to avoid
- Don't claim you "paused features to refactor" (rarely viable)
- Don't skip quantification (must show data)
- Don't forget guardrails (prevention matters)
- Don't skip outcomes (must show improvement)

## Self-assessment
- [ ] I can describe specific tech debt I addressed
- [ ] I can quantify the cost (incidents, time, velocity)
- [ ] I can explain how I prioritized (why this debt, not others)
- [ ] I can describe incremental approach
- [ ] I can quantify the improvement (incidents down, velocity up)

## Related Playbooks
- [Making Trade-off Decisions](../decision-making/making-trade-offs.md)
- [Prioritization and Saying No](../decision-making/prioritization-saying-no.md)
- [Continuous Learning](continuous-learning.md)

## Related Topics (knowledge base)
- [Testing Fundamentals](../../topics/quality/testing-fundamentals.md)
- [Code Quality Metrics](../../topics/quality/code-quality-metrics.md)
- (TODO) [Refactoring Strategies](../../topics/quality/refactoring-strategies.md)

## Notes / Refinements
- Boy-scout rule: leave code better than you found it
- Strategic debt is OK: shortcuts to ship fast, with plan to pay down
- Debt compounds: small today, crippling in 2 years
- Prevention >> cure: guardrails reduce new debt accumulation
