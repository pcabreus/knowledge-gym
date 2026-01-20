---
id: software-quality-assurance
title: "Software Quality Assurance (SQA)"
type: concept
status: learning
importance: 90
difficulty: medium
tags: [quality, testing, reliability, engineering-practices]
canonical: true
aliases: [SQA, quality-assurance]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Software Quality Assurance (SQA)

## TL;DR (BLUF)
- Software Quality Assurance is a systematic process to ensure software meets functional requirements and quality standards throughout development and operation.
- Use multiple quality dimensions: code quality, testing, operational quality, architecture, and process.
- Key trade-off: quality investment upfront vs. cost of failures later.

## Definition
**What it is:** A systematic set of practices, processes, and tools to prevent defects, ensure correctness, and maintain reliability throughout the software development lifecycle.  
**Key terms:** quality gates, defect prevention, test coverage, static analysis, SLI/SLO (Service Level Indicator/Objective), runbook, definition of done.

## Why it matters
- **Prevents production incidents:** Catching defects early is 10-100x cheaper than fixing them in production.
- **Enables fast delivery:** High quality reduces rework, making teams faster over time.
- **Builds trust:** Reliable systems allow engineers to move confidently and users to depend on the product.
- **Interview relevance:** Senior engineers must articulate quality strategies and trade-offs.

## Scope & Non-goals
**In scope:**
- Multi-dimensional quality: code, tests, operations, architecture, process.
- Preventive practices (gates, reviews, automation).
- Measurable quality signals (SLIs, error budgets, metrics).

**Out of scope / NOT solved by this:**
- Product-market fit or feature prioritization (quality ≠ "right features").
- Zero defects (impossible; manage acceptable failure rates with SLOs).

## Mental model / Intuition
Think of quality as **defense in depth**:
- **Layer 1 (Prevention):** Code reviews, linters, type systems.
- **Layer 2 (Detection):** Unit/integration/E2E tests catch issues before deploy.
- **Layer 3 (Mitigation):** Feature flags, canary deploys, circuit breakers limit blast radius.
- **Layer 4 (Response):** Observability, alerts, runbooks enable fast recovery.

No single layer is perfect; quality comes from the combination.

## Decision rules (When to use / When not to use)
### Use it when
- Building systems where failures have real cost (user impact, data loss, revenue).
- Working in teams (quality practices enable collaboration and trust).
- Operating long-lived systems (technical debt compounds without quality investment).

### Avoid it when
- Prototyping throwaway code for learning/validation (quality is overkill for experiments).
- Quality investment exceeds the value of the system (don't over-engineer a weekend project).

## How I would use it (practical)
- **Context:** Building a new API service in a team environment.
- **Steps:**
  1. Define quality gates: linters pass, tests >80% coverage, PR review required, CI green.
  2. Implement testing pyramid: many unit tests, fewer integration tests, critical E2E tests.
  3. Add observability: structured logs, metrics (latency p95/p99, error rate), alerts on SLO violations.
  4. Establish runbooks for common failure modes (DB timeout, rate limit exceeded).
  5. Use canary deploys: 5% traffic for 10 minutes before full rollout.
- **What success looks like:** Zero user-facing incidents in first month; P95 latency <200ms; error rate <0.1%.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Reduces production incidents and their cost.
  - Increases team velocity over time (less rework, safer refactoring).
  - Builds confidence to move fast.
- **Cons / Risks:**
  - Upfront cost: writing tests, code reviews, CI setup takes time.
  - Can slow initial delivery if over-applied (100% coverage on throwaway code).
  - Requires discipline and culture shift (not just tools).

### Alternatives
- **"Ship fast, fix bugs later":** Works for early MVPs but creates tech debt that compounds.
- **Manual QA only:** Doesn't scale; slow and misses edge cases.
- **Rely on monitoring alone:** You find defects in production instead of preventing them.

### How to choose
- **High stakes + long-lived system:** Invest in full SQA.
- **Early MVP / prototype:** Lightweight quality (basic tests + monitoring).
- **Team size matters:** Larger teams need stronger quality gates (more coordination risk).

## Failure modes & Pitfalls
- **Vanity metrics:** 90% test coverage means nothing if tests don't assert behavior or cover failure modes.
- **Too many gates:** Overly strict CI that blocks every small change kills velocity.
- **No ownership:** Quality is "QA team's job" instead of everyone's responsibility.
- **Testing the wrong thing:** E2E tests that are flaky and slow instead of fast, reliable unit tests.
- **Ignoring operational quality:** Perfect code that crashes under load or has no observability.

## Observability (How to detect issues)
- **Metrics:**
  - Lead time for changes (how long from commit to production).
  - Deployment frequency (higher is better with quality gates).
  - Change failure rate (% of deploys causing incidents).
  - Mean time to recovery (MTTR).
- **Logs:** Track defect sources (caught in CI, staging, production?).
- **Traces:** N/A for SQA itself (but critical for operational quality).
- **Alerts:** CI failures, test coverage drops below threshold, change failure rate spike.

## Implementation notes
- **Checklist**
  - [ ] Define "definition of done" (what quality means for your team).
  - [ ] Set up CI with quality gates (linters, tests, security scans).
  - [ ] Implement testing pyramid (many unit, fewer integration, critical E2E).
  - [ ] Add observability to production systems (logs, metrics, traces).
  - [ ] Establish SLIs/SLOs and error budgets.
  - [ ] Create runbooks for top 3 failure modes.
  - [ ] Require code reviews with checklist.
  - [ ] Use canary or blue/green deploys for risk mitigation.

- **Security / Compliance notes**
  - Include security scans in CI (SAST, dependency checks).
  - Track security defects separately (critical fixes don't wait for sprints).

- **Performance notes**
  - Add performance tests for critical paths (load testing under expected + peak traffic).
  - Monitor P95/P99 latency, not just averages.

- **Operational notes**
  - Quality doesn't end at deploy: monitor, alert, and iterate based on production behavior.
  - Blameless post-mortems after incidents improve the system.

## Mini example
```yaml
# Example CI quality gate (.github/workflows/quality.yml)
name: Quality Gates
on: [pull_request]
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Lint
        run: golangci-lint run
      - name: Unit tests
        run: go test ./... -race -coverprofile=coverage.out
      - name: Coverage check
        run: |
          coverage=$(go tool cover -func=coverage.out | grep total | awk '{print $3}' | sed 's/%//')
          if (( $(echo "$coverage < 80" | bc -l) )); then
            echo "Coverage $coverage% is below 80%"
            exit 1
          fi
      - name: Security scan
        run: gosec ./...
```

## Common anti-patterns
- **Anti-pattern:** "We'll add tests later" (they never get added; technical debt grows).
  - **Why it's bad:** Refactoring becomes risky; defects slip into production.
  - **Better approach:** Write tests as you code (TDD or test-soon-after-development).

- **Anti-pattern:** 100% code coverage mandate.
  - **Why it's bad:** Encourages meaningless tests that don't validate behavior.
  - **Better approach:** Focus coverage on critical paths and failure modes; accept lower coverage on simple getters/setters.

- **Anti-pattern:** No quality gates in CI ("trust developers to run tests locally").
  - **Why it's bad:** Humans forget; broken code reaches main branch.
  - **Better approach:** Automate gates in CI (linting, tests, security scans) that block merge.

## Interview readiness
### "Explain it like I'm teaching"
Software Quality Assurance is a multi-layered approach to prevent defects and ensure reliability. It combines code quality (reviews, linters, tests), operational quality (observability, SLOs, runbooks), and process discipline (CI gates, canary deploys). The goal isn't zero defects—it's to catch issues early and limit blast radius when failures occur. Quality is an investment: it costs time upfront but pays back in faster delivery and fewer production incidents.

### Trap questions (with answers)
1) **Q:** Doesn't high quality slow down delivery?
   - **A:** Initially yes, but quality accelerates delivery over time. Without tests and reviews, teams accumulate technical debt that requires constant firefighting and risky changes. Quality enables safe refactoring and confident iteration.

2) **Q:** If we have 90% test coverage, is our quality high?
   - **A:** Not necessarily. Coverage is a signal of what's *not* tested, but doesn't measure test quality. You can have 90% coverage with tests that don't assert behavior or cover failure modes. Focus on testing critical paths and edge cases, not hitting a coverage number.

3) **Q:** Should we fix all bugs before shipping?
   - **A:** No. Use risk-based prioritization: critical bugs (data loss, security) block releases; minor bugs (cosmetic issues) can ship with known trade-offs. Define your quality bar with SLOs—don't aim for zero defects, aim for acceptable failure rates.

4) **Q:** Who owns quality—developers or QA?
   - **A:** Everyone. Developers write tests and own code quality; QA (if present) designs test strategies and catches gaps. Quality is a team responsibility, not a handoff.

5) **Q:** How do you balance speed and quality?
   - **A:** Use risk-based trade-offs. For prototypes, lower quality is fine (no tests, minimal reviews). For production systems, invest in quality gates but keep them lean (fast tests, clear review checklists). Use feature flags and canary deploys to ship fast with limited blast radius.

### Quick self-check (5 items)
- [ ] I can define quality assurance across multiple dimensions (code, tests, operations, process).
- [ ] I can explain why quality accelerates delivery long-term despite upfront cost.
- [ ] I can name at least 3 quality gates and their purpose.
- [ ] I can articulate the difference between test coverage as a metric vs. quality.
- [ ] I can describe a failure mode (e.g., flaky tests, no observability) and how to prevent it.

## Links (NO duplication)
### Prerequisites
- [Reliability basics](../operations/reliability-basics.md)
- [Observability basics](../operations/observability-basics.md)
- [Testing fundamentals](testing-fundamentals.md)

### Related topics
- [Testing pyramid](testing-pyramid.md)
- [Code quality](code-quality.md)
- [Quality gates](quality-gates.md)
- [Clean code](clean-code.md)
- [Incident management basics](../operations/incident-management-basics.md)

### Compare with
- [Reliability basics](../operations/reliability-basics.md) — Reliability is the outcome; SQA is the process to achieve it.

## Notes / Inbox
- Consider adding examples from real incidents (leads leak, Heimdall failover loop) to illustrate quality gaps.
- Link to DORA metrics (lead time, deployment frequency, change failure rate, MTTR) when that topic is created.
