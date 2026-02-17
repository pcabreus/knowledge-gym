---
id: handling-conflict
title: "Handling Conflict with Peers"
type: playbook
category: behavioral
difficulty: hard
importance: 95
tags: [conflict, communication, teamwork, collaboration]
created_at: 2026-02-12
updated_at: 2026-02-12
---

# Handling Conflict with Peers

## Situation
**When this applies:**
- Disagreement with coworker on technical approach, priority, or decision
- Tension due to different working styles or communication preferences
- Conflict escalating or blocking progress

**Common manifestations:**
- "We should use approach A" vs "approach B is obviously better"
- Disagreement on what to prioritize (feature vs tech debt vs bug fix)
- Different interpretation of requirements or success criteria
- Clash between speed/quality, stability/innovation

## Why this matters
- Unresolved conflict wastes time, creates tension, blocks delivery
- How you handle disagreement signals maturity and leadership
- Critical interview topic: demonstrates collaboration, emotional intelligence, influence skills
- Teams judge you on how you disagree, not whether you agree

## My Process (Step-by-step)

### 1. Identify the type of conflict
- **What:** Diagnose root cause (objectives, priorities, data, style)
- **Why:** Different conflict types need different resolutions
- **How:** Ask yourself: do we disagree on the goal, the path, the data, or just communication style?
- **Example:** "We both want reliability, but I prioritize preventing incidents (pessimistic locking) and they prioritize throughput (optimistic locking)" → conflict of priorities, not goals

### 2. Reframe around shared objective
- **What:** Find the common ground (user impact, reliability, delivery, cost)
- **Why:** Shifts from "I'm right" to "what's best for the outcome"
- **How:** Start conversation with "we both want [shared goal]... how do we get there?"
- **Example:** "We both want this feature shipped on time without breaking prod. Let's figure out the safest fast path."

### 3. Request or bring evidence
- **What:** Ground the discussion in data, not opinions
- **Why:** Moves from subjective preference to objective analysis
- **How:** Ask for metrics, logs, incident history, benchmarks, cost projections, deadlines
- **Example:** "I pulled p99 latency from last month—this query is our top offender. If we optimize this, we solve 60% of user complaints."

### 4. Enumerate 2-3 options with trade-offs
- **What:** List alternatives explicitly with pros/cons
- **Why:** Clarifies the decision space and shows you're considering their view
- **How:** Write out options, each with time/risk/cost/quality implications
- **Example:**
  - **Option A (your preference):** Pessimistic lock → prevents overbooking, adds 50ms latency, simple to reason about
  - **Option B (their preference):** Optimistic locking → faster (no lock wait), retries under contention, complex retry logic
  - **Option C (hybrid):** Use optimistic for low-priority, pessimistic for critical paths

### 5. Propose decision with explicit criteria
- **What:** Recommend a choice and state what you're optimizing for
- **Why:** Makes reasoning transparent and allows informed disagreement
- **How:** "I recommend X because we optimize for [criterion], and I accept we sacrifice [other thing]"
- **Example:** "I recommend pessimistic lock because we optimize for correctness over latency, and our p99 is already 200ms so +50ms is acceptable."

### 6. Align on "disagree and commit"
- **What:** Get agreement on decision, owner, and next step—even if not unanimous
- **Why:** Prevents endless debate and ensures forward progress
- **How:** "Are we aligned? If not, let's escalate to [tech lead/PM]. Otherwise, I'll move forward with X by [date]."
- **Example:** "You disagree but we're blocked. I'll own this, ship pessimistic locking this sprint, measure latency impact, and we revisit if p99 degrades >10%."

### 7. Document to avoid re-debating
- **What:** Write decision, rationale, and trade-offs (mini-ADR / ticket comment)
- **Why:** Prevents repeating the same argument later
- **How:** 3-5 lines: decision, why, what we traded, how we'll measure
- **Example:** "Decision: pessimistic lock for seat reservation. Rationale: correctness over latency. Trade-off: +50ms p99. Metric: watch p99 <300ms."

### 8. Review after: what did we learn?
- **What:** Check if decision worked; capture learnings
- **Why:** Builds trust and improves future decisions
- **How:** After 1-2 weeks, look at metrics and ask "was this the right call? What would we do differently?"
- **Example:** "Latency stayed flat (good). Retries dropped to zero (great). Next time: use pessimistic lock earlier for inventory-type resources."

## Success Signals
**You know it's working when:**
- [ ] Conflict resolved in 1-2 conversations, not weeks of tension
- [ ] Decision made with clear owner and next steps
- [ ] No lingering resentment or passive-aggressive behavior
- [ ] Colleague collaborates smoothly on next project
- [ ] Team cites your documented decision later

## Common Pitfalls (and how to avoid them)

### Pitfall 1: Digging into positions instead of interests
- **Why it happens:** Ego, sunk cost, fear of "losing"
- **Impact:** Endless debate, team frustration, escalation
- **Instead do:** Ask "what outcome do you want?" and "why?" until you find shared goal

### Pitfall 2: Avoiding conflict (hoping it resolves itself)
- **Why it happens:** Discomfort, conflict-averse culture
- **Impact:** Festering tension, passive-aggressive behavior, missed deadlines
- **Instead do:** Address early and directly with curiosity, not aggression

### Pitfall 3: Escalating too quickly
- **Why it happens:** Impatience, lack of trust in peer resolution
- **Impact:** Undermines colleague, damages relationship, reputation as unable to collaborate
- **Instead do:** Try 1-2 direct conversations first; escalate only if blocked or critical impact

### Pitfall 4: Not documenting the decision
- **Why it happens:** Urgency, assuming "we'll remember"
- **Impact:** Same debate re-emerges weeks later
- **Instead do:** Spend 5 minutes writing decision and rationale immediately

## Real-world Example
**Context:** Teammate wanted to rewrite auth service in new framework; I wanted to patch critical bug in existing system first.

**What happened:** We debated for 2 days, blocking both tasks.

**Process applied:**
1. Identified conflict type: priority, not approach
2. Reframed: "We both want secure, maintainable auth. What's the user impact timeline?"
3. Brought evidence: 5 security incidents in 30 days; rewrite is 3 weeks away
4. Enumerated options: (A) patch now + rewrite later, (B) rewrite now and accept risk, (C) workaround + fast rewrite
5. Proposed: "Optimize for security over velocity; patch this week, rewrite next sprint"
6. Disagree and commit: They disagreed but committed; I owned the patch
7. Documented in ticket with timeline
8. Reviewed: Patch worked, no incidents, rewrite shipped 2 weeks later

**Outcome:** Bug fixed in 2 days, no incidents, rewrite completed safely. Colleague appreciated data-driven approach.

## Interview Tips
### How to talk about this
- **Framework to use:** "Shared goal → evidence → options → decision → commit"
- **Key points to emphasize:** Data-driven, collaborative, outcome-focused, ownership
- **Language that works:** "aligned on outcome," "enumerated trade-offs," "disagree and commit," "measured results"

### Sample STAR response outline
- **Situation:** "Disagreed with senior engineer on [technical approach] for [critical system]"
- **Task:** "Needed to resolve quickly to unblock [deadline/user impact]"
- **Action:** "Reframed around shared goal, brought [specific evidence], enumerated 3 options with trade-offs, proposed decision optimizing for [criterion], aligned on disagree-and-commit"
- **Result:** "Shipped in [time], measured [metric], no incidents, improved team collaboration on next project"

### Red flags to avoid
- Don't blame: "they were being stubborn"
- Don't avoid ownership: "we couldn't agree so it didn't happen"
- Don't skip the outcome: must show resolution and impact
- Don't hide the disagreement: interviewers want to see you handle conflict

## Self-assessment
- [ ] I can describe a specific conflict I resolved (not avoided)
- [ ] I can name the shared goal we aligned on
- [ ] I can list the 2-3 options I enumerated
- [ ] I can state the decision criteria explicitly
- [ ] I can quantify the outcome (time saved, metric improved, relationship maintained)

## Related Playbooks
- [Influencing Without Authority](influencing-without-authority.md)
- [Making Trade-off Decisions](../decision-making/making-trade-offs.md)
- [Prioritization and Saying No](../decision-making/prioritization-saying-no.md)

## Related Topics (knowledge base)
- (TODO) [Communication Patterns](../topics/soft-skills/communication-patterns.md)
- (TODO) [Decision Records (ADR)](../topics/architecture/decision-records.md)

## Notes / Refinements
- Works best when both parties are acting in good faith
- Escalate if conflict involves ethics, safety, or legal concerns
- Cultural differences matter: some cultures value direct disagreement, others prefer indirect
- Adapt tone: senior colleague needs more deference than peer
