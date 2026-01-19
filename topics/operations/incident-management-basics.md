---
id: incident-management-basics
title: "Incident Management Basics"
type: skill
status: learning
importance: 40
difficulty: medium
tags: [operations, reliability]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Incident Management Basics

## TL;DR (BLUF)
- Incident management is the process of detecting, triaging, and resolving outages.
- Use runbooks and clear roles to reduce MTTR.
- Trade-off: process overhead vs chaos during outages.

## Definition
**What it is:** Structured response to production incidents.
**Key terms:** MTTR, on-call, runbook, postmortem.

## Why it matters
- It reduces downtime and user impact.

## Scope & Non-goals
**In scope:** incident response fundamentals.
**Out of scope / NOT solved by this:** deep SRE practices.

## Mental model / Intuition
- Incidents need coordination more than heroics.

## Decision rules (When to use / When not to use)
### Use it when
- There is user-impacting downtime or degradation.
### Avoid it when
- The issue is cosmetic or non-user-impacting.

## How I would use it (practical)
- **Context:** Latency spike in production.
- **Steps:** declare incident → mitigate → communicate → postmortem.
- **What success looks like:** fast recovery and documented learnings.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** faster resolution and clarity.
- **Cons / Risks:** process overhead.
### Alternatives
- **Ad-hoc response:** faster initially but inconsistent.
- **How to choose:** use formal incidents for user impact.

## Failure modes & Pitfalls
- Poor communication leading to duplicated work.

## Observability (How to detect issues)
- **Metrics:** time to detect, time to recover.
- **Logs:** incident timelines.
- **Alerts:** repeated incidents without follow-up.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define on-call roles
  - [ ] Maintain runbooks

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** No postmortems.
  - **Why it’s bad:** repeated incidents.
  - **Better approach:** run blameless postmortems.

## Interview readiness
### “Explain it like I’m teaching”
- Incident management is a structured way to handle outages: detect, triage, mitigate, and learn. It reduces MTTR and prevents repeat issues.

### Trap questions (with answers)
1) **Q:** Are postmortems optional?
   - **A:** no; they prevent recurrence.
2) **Q:** Should one person fix everything?
   - **A:** no; coordinated roles work best.
3) **Q:** Is communication part of incident response?
   - **A:** yes; it’s critical.

### Quick self-check (5 items)
- [ ] I can define incident management.
- [ ] I can describe the lifecycle.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain MTTR.

## Links (NO duplication)
### Prerequisites
- [Observability basics](observability-basics.md)

### Related topics
- [Reliability basics](reliability-basics.md)

### Compare with
- [Operational planning](operational-planning.md) — prevention vs response.
