---
id: requirements-basics
title: "Requirements Basics"
type: skill
status: learning
importance: 40
difficulty: easy
tags: [system-design]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Requirements Basics

## TL;DR (BLUF)
- Requirements define what a system must do and constraints.
- Use them to guide design and trade-offs.
- Trade-off: too much detail can slow progress.

## Definition
**What it is:** The process of capturing functional and non-functional needs.
**Key terms:** functional requirements, non-functional requirements, constraints.

## Why it matters
- Clear requirements reduce rework.
- Ambiguity leads to wrong architecture choices.

## Scope & Non-goals
**In scope:** capturing and prioritizing requirements.
**Out of scope / NOT solved by this:** detailed product management.

## Mental model / Intuition
- Requirements are the contract for what the system must achieve.

## Decision rules (When to use / When not to use)
### Use it when
- You start a new system or major change.
### Avoid it when
- You’re making a trivial change.

## How I would use it (practical)
- **Context:** New data pipeline.
- **Steps:** define goals → identify constraints → document trade-offs.
- **What success looks like:** aligned design decisions.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** clarity.
- **Cons / Risks:** analysis paralysis.
### Alternatives
- **Prototype-first:** faster learning.
- **How to choose:** use requirements when correctness and scale matter.

## Failure modes & Pitfalls
- Overlooking non-functional requirements.

## Observability (How to detect issues)
- **Metrics:** scope churn, change requests.
- **Logs:** requirement changes.
- **Alerts:** repeated rework signals.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Capture functional needs
  - [ ] Capture non-functional constraints

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Designing without clarity on goals.
  - **Why it’s bad:** wasted effort.
  - **Better approach:** document requirements early.

## Interview readiness
### “Explain it like I’m teaching”
- Requirements are the baseline for design: what must the system do and what constraints exist. They prevent misaligned architecture decisions.

### Trap questions (with answers)
1) **Q:** Are requirements only functional?
   - **A:** no; non-functional constraints are critical.
2) **Q:** Are requirements fixed forever?
   - **A:** no; they evolve but should be tracked.
3) **Q:** Can you skip requirements for small changes?
   - **A:** sometimes, but be explicit.

### Quick self-check (5 items)
- [ ] I can define requirements.
- [ ] I can name non-functional constraints.
- [ ] I can describe a pitfall.
- [ ] I can explain a trade-off.
- [ ] I can connect requirements to design.

## Links (NO duplication)
### Prerequisites
- N/A

### Related topics
- [API design basics](api-design-basics.md)

### Compare with
- [Performance basics](performance-basics.md) — constraints vs optimization.
