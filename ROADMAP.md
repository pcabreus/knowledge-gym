# ROADMAP — Knowledge Gym

## Goal
Build a personal system (and potentially a product) to master technical knowledge with deep learning + long-term retention.

---

## Phase 1 — Static base (Markdown) ✅
**Goal:** have a navigable topic graph with high quality.

Deliverables:
- [ ] Topic taxonomy and conventions
- [ ] Topic template + authoring guide
- [ ] Prompts for:
  - creating topics
  - generating MCQ questions
  - evaluating open answers
- [ ] 30–50 core topics (Go, DB, system design, resiliency)
- [ ] 10 real cases (STAR/BLUF) as interview material

KPIs:
- Navigability (correct links)
- Zero duplication
- Core topics with “when/when not + trade-offs + traps”

---

## Phase 2 — Routine and practice (semi-automated)
**Goal:** practice without friction.

Ideas:
- [ ] Question sets by area (Go, DB, distributed systems, interviews)
- [ ] Manual “daily drill”: 10 questions + 1 teach-back
- [ ] Weekly review as a mock interview

KPIs:
- Weekly consistency
- Question coverage per core topic

---

## Phase 3 — Gamification (App + DB)
**Goal:** automatic scheduling + tracking.

Ideas:
- [ ] DB for attempts / scores / next_review_at
- [ ] Spaced repetition (SM-2)
- [ ] Living importance (goes up with interviews/projects, down with disuse)
- [ ] Simple UI (web) or CLI/TUI

KPIs:
- Retention measured by score at 30/60/90 days
- Reduce gaps (topics with low retention)

---

## Phase 4 — Product (optional)
**Goal:** turn it into a tool usable by others.

Ideas:
- [ ] Import/export
- [ ] Templates by role (Backend Go, SRE, etc.)
- [ ] Packs for interviews/certifications

KPIs:
- Time to “interview-ready”
- User feedback
