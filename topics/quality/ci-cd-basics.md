---
id: ci-cd-basics
title: "CI/CD Basics"
type: concept
status: learning
importance: 85
difficulty: medium
tags: [devops, automation, deployment, integration, delivery]
canonical: true
aliases: [continuous-integration, continuous-deployment, continuous-delivery]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# CI/CD Basics

## TL;DR (BLUF)
- CI/CD automates code integration, testing, and deployment to reduce manual errors and accelerate delivery.
- CI (Continuous Integration): merge code frequently, run automated tests.
- CD: Continuous Delivery (ready to deploy) or Continuous Deployment (auto-deploy to production).
- Key trade-off: automation setup cost vs. delivery speed and reliability.

## Definition
**What it is:** A set of practices and tools that automate the process of integrating code changes, testing them, and deploying to production.  
**Key terms:**
- **CI (Continuous Integration):** Merging code changes frequently (daily/hourly) with automated builds and tests.
- **CD (Continuous Delivery):** Code is always in a deployable state; deployment requires manual approval.
- **CD (Continuous Deployment):** Every change that passes tests is automatically deployed to production.
- **Pipeline:** Automated sequence of steps (build, test, deploy).
- **Artifact:** Built package ready for deployment (Docker image, binary, etc.).

## Why it matters
- **Faster delivery:** Automated pipelines deploy in minutes vs. hours/days for manual processes.
- **Fewer bugs:** Automated tests catch defects before production.
- **Reduces risk:** Small, frequent releases are easier to debug and rollback than large releases.
- **Better collaboration:** CI forces integration early, catching conflicts quickly.
- **Interview relevance:** CI/CD is fundamental for modern software delivery discussions.

## Scope & Non-goals
**In scope:**
- CI: automated builds, tests, integration.
- CD: deployment automation, staging/production pipelines.
- Tools: GitHub Actions, Jenkins, GitLab CI, CircleCI (conceptually).

**Out of scope / NOT solved by this:**
- Infrastructure-as-Code (IaC) details (Terraform, CloudFormation—separate topic).
- Specific cloud platforms (AWS, GCP, Azure—separate topics).

## Mental model / Intuition
Think of CI/CD as an **assembly line for software**:
- **CI:** Quality control at each station (automated tests catch defects early).
- **CD (Delivery):** Product ready to ship (but waits for manual approval).
- **CD (Deployment):** Product ships automatically when ready (no manual gate).

Without CI/CD, you're hand-crafting each release (slow, error-prone, risky).

## Decision rules (When to use / When not to use)
### Use it when
- Building production systems with frequent changes.
- Working in teams (CI catches integration conflicts early).
- Deploying frequently (daily or more).
- Quality gates are necessary (automated tests before deploy).

### Avoid it when
- Writing throwaway code (prototypes, one-off scripts).
- Deployment is extremely risky (safety-critical systems may require manual checks; though CI/CD with gates is still valuable).

## How I would use it (practical)
- **Context:** Setting up CI/CD for a Go API service deployed to AWS.
- **Steps:**
  1. **CI (on every PR):**
     - Trigger: Pull request opened.
     - Steps: Checkout code → Install dependencies → Lint → Run unit tests → Check coverage.
     - Outcome: Block merge if linting fails or tests fail.
  2. **CD (on merge to main):**
     - Trigger: Code merged to main branch.
     - Steps: Build Docker image → Run integration tests → Push image to registry → Tag with version.
     - Outcome: Artifact ready for deployment.
  3. **CD (deploy to staging):**
     - Trigger: New image in registry.
     - Steps: Deploy to staging environment → Run smoke tests → Monitor for 10 minutes.
     - Outcome: Staging environment updated automatically.
  4. **CD (deploy to production):**
     - Option A (Continuous Delivery): Manual approval required.
     - Option B (Continuous Deployment): Auto-deploy if staging succeeds + canary deploy (5% traffic for 10 min).
- **What success looks like:** Code goes from commit to production in <30 minutes; zero manual deployment steps; error rate <0.1%.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Faster delivery (automate repetitive tasks).
  - Fewer defects (automated tests catch bugs early).
  - Reduced risk (small, frequent releases).
  - Better visibility (pipeline shows build/test status).
- **Cons / Risks:**
  - Upfront setup cost (writing pipelines, configuring tools).
  - Requires discipline (tests must be reliable, no flaky tests).
  - Initial learning curve (teams need CI/CD knowledge).

### Alternatives
- **Manual builds/deployments:** Slow, error-prone, doesn't scale.
- **Scheduled releases (weekly/monthly):** Fewer deployments but larger, riskier releases.
- **No testing automation:** Faster initially but high defect rate in production.

### How to choose
- **Production systems:** Full CI/CD with automated tests and deployments.
- **Internal tools:** Lightweight CI (tests) + manual deploys may suffice.
- **Prototypes:** No CI/CD (overkill for throwaway code).

## Failure modes & Pitfalls
- **Slow pipelines:** CI takes >30 minutes; developers context-switch and lose productivity.
- **Flaky tests:** Random test failures block deployments; teams bypass CI or ignore failures.
- **No rollback strategy:** Deployment breaks production with no quick way to revert.
- **Deploy everything to production:** No staging environment to catch issues before prod.
- **Over-complicated pipelines:** Too many stages, manual approvals, or dependencies (kills velocity).

## Observability (How to detect issues)
- **Metrics:**
  - Pipeline duration (flag >30 minutes for PRs, >60 minutes for deploys).
  - Pipeline success rate (aim for >95%).
  - Deployment frequency (higher is better with CI/CD).
  - Mean time to recovery (MTTR; how fast can you rollback?).
- **Logs:** Track pipeline failures by stage (build vs test vs deploy).
- **Traces:** N/A for CI/CD itself.
- **Alerts:** Pipeline duration spike, success rate drop, deployment failure.

## Implementation notes
- **Checklist**
  - [ ] Set up CI for PRs (lint, unit tests, coverage check).
  - [ ] Set up CD for main branch (build artifact, integration tests).
  - [ ] Define deployment stages (staging → production).
  - [ ] Implement rollback strategy (canary, blue/green, or manual revert).
  - [ ] Monitor pipeline metrics (duration, success rate).
  - [ ] Fix or quarantine flaky tests immediately.
  - [ ] Use secrets management (don't hardcode credentials in pipelines).

- **Security / Compliance notes**
  - Include security scans in CI (SAST, dependency checks).
  - Use least-privilege credentials for deployment (IAM roles, service accounts).
  - Audit deployments (who deployed what, when).

- **Performance notes**
  - Parallelize pipeline steps where possible (lint + test in parallel).
  - Cache dependencies (Go modules, npm packages) to speed up builds.
  - Run expensive tests (E2E, load tests) less frequently (main branch only, nightly).

- **Operational notes**
  - Monitor production after deployments (error rate, latency spikes).
  - Automate rollback if metrics degrade (canary with auto-revert).

## Mini example (GitHub Actions)
```yaml
# .github/workflows/ci.yml
name: CI
on:
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      - name: Run linter
        run: golangci-lint run --timeout 5m

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      - name: Run tests
        run: go test ./... -race -coverprofile=coverage.out
      - name: Check coverage
        run: |
          coverage=$(go tool cover -func=coverage.out | grep total | awk '{print $3}' | sed 's/%//')
          if (( $(echo "$coverage < 80" | bc -l) )); then
            echo "Coverage $coverage% is below 80%"
            exit 1
          fi

# .github/workflows/cd.yml
name: CD
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      - name: Push to registry
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker push myapp:${{ github.sha }}

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: |
          # Deploy to staging environment (kubectl, AWS CLI, etc.)
          kubectl set image deployment/myapp myapp=myapp:${{ github.sha }} -n staging
      - name: Smoke tests
        run: |
          # Run basic smoke tests against staging
          curl -f https://staging.example.com/health || exit 1
```

## Common anti-patterns
- **Anti-pattern:** No CI pipeline ("we'll test locally").
  - **Why it's bad:** Human error; tests skipped; broken code reaches main.
  - **Better approach:** Automate tests in CI; block merge on failures.

- **Anti-pattern:** Deploy directly to production (no staging).
  - **Why it's bad:** Issues discovered in production; high blast radius.
  - **Better approach:** Deploy to staging first; validate before production.

- **Anti-pattern:** Manual deployment steps.
  - **Why it's bad:** Slow, error-prone, not repeatable.
  - **Better approach:** Automate deployment with scripts or tools (kubectl, AWS CLI).

- **Anti-pattern:** Ignoring flaky tests ("just re-run CI").
  - **Why it's bad:** Erodes trust in CI; developers bypass or ignore failures.
  - **Better approach:** Fix or quarantine flaky tests immediately.

## Interview readiness
### "Explain it like I'm teaching"
CI/CD automates the path from code commit to production. CI means merging code frequently and running automated tests to catch bugs early. CD has two flavors: Continuous Delivery means code is always deployable but requires manual approval to go to production, while Continuous Deployment means every change that passes tests is automatically deployed. The key benefits are speed (automate repetitive tasks), quality (automated tests catch defects), and reduced risk (small, frequent releases are easier to debug and rollback). A typical pipeline runs linting and unit tests on every PR, builds artifacts and runs integration tests on main, and deploys to staging and production with appropriate gates.

### Trap questions (with answers)
1) **Q:** What's the difference between Continuous Delivery and Continuous Deployment?
   - **A:** 
     - **Continuous Delivery:** Code is always in a deployable state, but deployment to production requires manual approval.
     - **Continuous Deployment:** Every change that passes automated tests is deployed to production automatically (no manual gate).
     - Continuous Deployment is more automated but requires higher confidence in tests.

2) **Q:** Doesn't CI/CD mean deploying untested code to production?
   - **A:** No. CI/CD relies on automated tests at every stage. Code that fails tests is blocked from merging or deploying. The goal is to automate *testing* and *deployment*, not to skip testing.

3) **Q:** How do you prevent bad deployments in Continuous Deployment?
   - **A:** Use progressive rollout strategies:
     - **Canary deploys:** Deploy to small percentage of traffic first; rollback if errors spike.
     - **Feature flags:** Deploy code but keep features off; enable gradually.
     - **Automated rollback:** Monitor metrics (error rate, latency); auto-revert if thresholds breached.

4) **Q:** What if CI pipeline is too slow (>1 hour)?
   - **A:** Optimize:
     - Parallelize steps (run lint + test in parallel).
     - Cache dependencies (Go modules, npm packages).
     - Tier tests (fast tests on PR; slow tests on main).
     - Use faster hardware or runners.
     - Target: <10 minutes for PRs, <30 minutes for full pipeline.

5) **Q:** Should every project have CI/CD?
   - **A:** Not necessarily. Use CI/CD for production systems with frequent changes and multiple engineers. For prototypes, one-off scripts, or stable systems with rare releases, the setup cost may exceed the benefit. But for most modern software, CI/CD is essential.

### Quick self-check (5 items)
- [ ] I can explain the difference between CI, Continuous Delivery, and Continuous Deployment.
- [ ] I can describe a typical CI/CD pipeline (stages and gates).
- [ ] I can name at least 3 benefits of CI/CD (speed, quality, risk reduction).
- [ ] I can explain strategies to prevent bad deployments (canary, feature flags, rollback).
- [ ] I can articulate when CI/CD is overkill (and when it's essential).

## Links (NO duplication)
### Prerequisites
- [Software Quality Assurance](software-quality-assurance.md)
- [Testing fundamentals](testing-fundamentals.md)

### Related topics
- [Quality gates](quality-gates.md)
- [Testing pyramid](testing-pyramid.md)
- [Infrastructure as Code](../operations/infrastructure-as-code.md)
- [Blue/green deployments](../operations/blue-green-deployments.md)
- [Canary deployments](../operations/canary-deployments.md)

### Compare with
- [Quality gates](quality-gates.md) — Quality gates are the checkpoints within a CI/CD pipeline that enforce quality standards.

## Notes / Inbox
- Add examples from real projects: Heimdall CI/CD setup, Opportunity Actions deployment pipeline.
- Consider adding section on CI/CD tools comparison (GitHub Actions vs Jenkins vs GitLab CI).
- Link to deployment strategies (blue/green, canary, feature flags) when those topics are created.
