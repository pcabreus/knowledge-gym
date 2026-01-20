---
id: quality-gates
title: "Quality Gates (CI/CD)"
type: pattern
status: learning
importance: 80
difficulty: medium
tags: [quality, ci-cd, automation, deployment]
canonical: true
aliases: [ci-cd-gates, build-gates]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Quality Gates (CI/CD)

## TL;DR (BLUF)
- Quality gates are automated checks in CI/CD pipelines that prevent low-quality code from reaching production.
- Use gates for linting, testing, security scans, and coverage thresholds.
- Key trade-off: preventing defects vs. pipeline speed and developer friction.

## Definition
**What it is:** Automated checkpoints in a CI/CD pipeline that validate code quality, security, and correctness before allowing code to merge or deploy.  
**Key terms:** CI (Continuous Integration), CD (Continuous Deployment/Delivery), linter, static analysis, coverage threshold, security scan (SAST/DAST), smoke test.

## Why it matters
- **Prevents defects:** Catches issues before they reach production (bugs, security vulnerabilities, style violations).
- **Enforces standards:** Ensures all code meets minimum quality bar without relying on human memory.
- **Enables fast delivery:** Teams ship confidently because gates catch regressions automatically.
- **Interview relevance:** CI/CD and quality automation are standard topics in system design and DevOps discussions.

## Scope & Non-goals
**In scope:**
- Pre-merge gates (linting, unit tests, coverage).
- Pre-deploy gates (integration tests, security scans, smoke tests).
- Post-deploy gates (canary metrics, smoke tests in production).

**Out of scope / NOT solved by this:**
- Manual QA or exploratory testing (complements gates).
- Product quality (gates validate technical quality, not feature correctness).

## Mental model / Intuition
Think of quality gates as **airport security checkpoints**:
- You can't board the plane (deploy to production) without passing security (gates).
- Multiple layers: ID check (linting), baggage scan (tests), body scan (security scans).
- If you fail a check, you can't proceed until you fix it.

Without gates, defective code slips through and causes production incidents.

## Decision rules (When to use / When not to use)
### Use it when
- Building production systems where defects have real cost.
- Working in teams (gates enforce shared standards).
- Deploying frequently (gates enable safe, fast releases).

### Avoid it when
- Prototyping throwaway code (gates add friction for no value).
- Gates are too slow (>30 minutes for CI kills productivity; optimize or relax gates).

## How I would use it (practical)
- **Context:** Building a Go API service deployed via GitHub Actions.
- **Steps:**
  1. **Pre-commit hooks (local):**
     - Run linter (`golangci-lint`) and formatters (`gofmt`).
     - Block commit if linting fails (catches issues before PR).
  2. **Pre-merge gates (CI on PR):**
     - Linting: `golangci-lint run --timeout 5m`
     - Unit tests: `go test ./... -race -coverprofile=coverage.out`
     - Coverage check: Fail if coverage <80%.
     - Security scan: `gosec ./...` (catch SQL injection, unsafe code).
     - Static analysis: `go vet`, `staticcheck`.
  3. **Pre-deploy gates (CI on main branch):**
     - Integration tests: Run against Docker containers (PostgreSQL, Redis).
     - Smoke tests: Deploy to staging, run critical API calls.
     - Container security scan: `trivy` on Docker images.
  4. **Post-deploy gates (production):**
     - Canary deploy: 5% traffic for 10 minutes.
     - Monitor error rate and P95 latency.
     - Auto-rollback if error rate >0.5%.
- **What success looks like:** CI completes in <10 minutes; zero production incidents from preventable defects; PR merge blocked on test failures.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Prevents defects from reaching production.
  - Automates enforcement (no reliance on human memory).
  - Builds confidence to move fast.
- **Cons / Risks:**
  - Slow gates kill productivity (developers wait for CI).
  - Overly strict gates create friction (minor style violations block PRs).
  - False positives (flaky tests, overly aggressive security scans) erode trust.

### Alternatives
- **Manual code review only:** Catches some issues but misses what tools automate (unused variables, security risks).
- **No gates ("trust developers"):** Fast initially but defects slip through; production incidents increase.
- **Post-deploy gates only:** Ship fast but catch defects in production instead of CI (higher blast radius).

### How to choose
- **High-stakes production system:** Use full gates (linting, tests, security, canary deploys).
- **Internal tools / prototypes:** Lightweight gates (basic linting, no coverage mandates).
- **Balance speed:** Keep gates fast (<10 minutes for PR CI); run slower tests (E2E, load tests) on main branch or nightly.

## Failure modes & Pitfalls
- **Slow gates:** CI takes 30+ minutes; developers context-switch and lose productivity.
- **Flaky tests:** Random failures erode trust; developers bypass gates or re-run until they pass.
- **Overly strict rules:** Minor style violations block PRs; kills velocity and morale.
- **Security theater:** Scanning for vulnerabilities but not fixing them (alerts ignored).
- **No incremental gates:** Running all tests on every commit; should run unit tests on PR, integration tests on main.

## Observability (How to detect issues)
- **Metrics:**
  - CI duration (per pipeline stage; flag >10 minutes for PRs).
  - Gate pass rate (% of builds passing first try; aim for >95%).
  - Flakiness rate (% of tests failing randomly; aim for <1%).
  - Blocked PRs (how many PRs waiting on CI; flag >5).
- **Logs:** Track gate failures by type (linting vs tests vs security).
- **Traces:** N/A for gates themselves.
- **Alerts:** CI duration spike, flakiness rate increase, security scan failures.

## Implementation notes
- **Checklist**
  - [ ] Set up CI pipeline (GitHub Actions, Jenkins, GitLab CI).
  - [ ] Add linting gate (fail on violations).
  - [ ] Add unit test gate (fail on test failures or coverage drop).
  - [ ] Add security scan (SAST for code, DAST for APIs, container scan for images).
  - [ ] Add integration test gate on main branch (not on every PR to save time).
  - [ ] Implement canary deploys with auto-rollback.
  - [ ] Monitor gate metrics (duration, pass rate, flakiness).
  - [ ] Quarantine or fix flaky tests immediately.

- **Security / Compliance notes**
  - Include SAST (static analysis), DAST (dynamic testing), and dependency scans (Snyk, Dependabot).
  - Block deploys on critical vulnerabilities (don't just warn).

- **Performance notes**
  - Parallelize test execution to speed up CI.
  - Use caching for dependencies (Go modules, npm packages).
  - Run expensive tests (E2E, load tests) on main branch or nightly, not every PR.

- **Operational notes**
  - Gates are not "set and forget": monitor false positives and adjust thresholds.
  - Fast feedback is critical: optimize slow gates or run them less frequently.

## Mini example
```yaml
# GitHub Actions quality gates example
name: Quality Gates
on:
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      - name: Run linter
        run: golangci-lint run --timeout 5m

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      - name: Run unit tests
        run: go test ./... -race -coverprofile=coverage.out
      - name: Check coverage
        run: |
          coverage=$(go tool cover -func=coverage.out | grep total | awk '{print $3}' | sed 's/%//')
          if (( $(echo "$coverage < 80" | bc -l) )); then
            echo "Coverage $coverage% is below 80%"
            exit 1
          fi

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run security scan
        run: |
          go install github.com/securego/gosec/v2/cmd/gosec@latest
          gosec ./...

  integration-tests:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'  # Only on main branch
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: testpass
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v3
      - name: Run integration tests
        run: go test ./integration/... -tags=integration
```

## Common anti-patterns
- **Anti-pattern:** No gates ("we'll catch bugs in code review").
  - **Why it's bad:** Humans are inconsistent; tools never forget.
  - **Better approach:** Automate what can be automated (linting, tests, security scans); reserve human review for design and logic.

- **Anti-pattern:** Gates that take >30 minutes.
  - **Why it's bad:** Developers lose context, productivity drops, frustration increases.
  - **Better approach:** Parallelize tests, use caching, run expensive tests less frequently (main branch only or nightly).

- **Anti-pattern:** Ignoring flaky tests ("just re-run CI").
  - **Why it's bad:** Erodes trust in gates; developers bypass or ignore failures.
  - **Better approach:** Quarantine flaky tests immediately; fix root cause (race conditions, timeouts) or delete.

- **Anti-pattern:** Too many gates on every PR.
  - **Why it's bad:** Slows feedback loops; kills productivity.
  - **Better approach:** Tier gates—fast gates on PRs (lint, unit tests), slower gates on main (integration, E2E, load tests).

## Interview readiness
### "Explain it like I'm teaching"
Quality gates are automated checks in CI/CD that prevent bad code from reaching production. They run at different stages: linting and unit tests on every PR, integration tests on the main branch, and canary deploys in production. The goal is to catch defects early and enforce quality standards automatically. Gates must be fast (sub-10 minutes for PRs) and reliable (no flaky tests) or they kill productivity. You tier gates based on cost—run cheap, fast tests on every change; run expensive, slow tests less frequently.

### Trap questions (with answers)
1) **Q:** Don't quality gates slow down delivery?
   - **A:** Only if implemented poorly. Fast gates (<10 minutes) actually accelerate delivery by catching defects before they reach production, reducing rework and incident response time. Slow or flaky gates do hurt productivity—optimize or tier them.

2) **Q:** What's the most important quality gate?
   - **A:** No single gate is most important. Use layers: linting catches style/correctness issues, tests catch logic bugs, security scans catch vulnerabilities. Each gate prevents different failure modes. If forced to choose, tests are highest impact (catch most bugs).

3) **Q:** Should gates be 100% strict (never allow exceptions)?
   - **A:** No. Use risk-based exceptions: security vulnerabilities should always block, but minor linting violations might be acceptable in emergencies. Track exceptions and fix them later. Don't make gates so strict they encourage bypassing.

4) **Q:** How do you handle flaky tests in gates?
   - **A:** Quarantine immediately (skip them or move to a separate suite). Flaky tests erode trust—developers will bypass gates or ignore failures. Fix root cause (race conditions, environmental dependencies) or delete. Never tolerate flakiness.

5) **Q:** Should you run all tests on every PR?
   - **A:** No. Tier tests: fast unit tests on every PR (<5 minutes), integration tests on main branch (<10 minutes), E2E and load tests nightly or on releases. Running all tests on every PR makes CI too slow and kills productivity.

### Quick self-check (5 items)
- [ ] I can name at least 4 types of quality gates and their purpose.
- [ ] I can explain the trade-off between gate strictness and developer velocity.
- [ ] I can describe how to tier gates (PR vs main branch vs production).
- [ ] I can articulate why flaky tests are unacceptable in gates.
- [ ] I can give an example of when to relax a gate (risk-based exception).

## Links (NO duplication)
### Prerequisites
- [Software Quality Assurance](software-quality-assurance.md)
- [Testing pyramid](testing-pyramid.md)
- [CI/CD basics](ci-cd-basics.md)

### Related topics
- [Code quality](code-quality.md)
- [Clean code](clean-code.md)

### Compare with
- [Manual code review](../operations/operational-planning.md) — Gates automate what can be automated; reviews focus on design and logic.

## Notes / Inbox
- Add examples from real projects: Heimdall CI setup, Opportunity Actions security scans.
- Consider adding section on post-deploy gates (canary deploys, feature flags, monitoring).
