---
id: code-quality
title: "Code Quality"
type: concept
status: learning
importance: 85
difficulty: medium
tags: [quality, code-review, static-analysis, maintainability]
canonical: true
aliases: [code-quality-metrics]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Code Quality

## TL;DR (BLUF)
- Code quality measures how readable, maintainable, and correct code is, using metrics like complexity, test coverage, and static analysis findings.
- Use automated tools (linters, code review) combined with manual review to maintain quality.
- Key trade-off: time spent on quality improvements vs. shipping features.

## Definition
**What it is:** A multi-dimensional assessment of source code based on correctness, readability, maintainability, testability, and adherence to standards.  
**Key terms:** cyclomatic complexity, code coverage, static analysis, linter, code review, technical debt, SOLID principles, code smell.

## Why it matters
- **Maintainability:** High-quality code is easier to change, refactor, and debug.
- **Velocity:** Teams move faster in clean codebases; low-quality code creates drag (bug fixes, confusion, fear of breaking things).
- **Onboarding:** New engineers ramp up faster when code is readable and well-structured.
- **Interview relevance:** Senior engineers must articulate trade-offs between code quality and speed, and defend design decisions.

## Scope & Non-goals
**In scope:**
- Automated quality signals: linters, complexity metrics, test coverage.
- Human quality signals: code reviews, readability, naming.
- Preventing common defects through static analysis.

**Out of scope / NOT solved by this:**
- Product quality (features working as intended; requires testing).
- Performance (separate concern, though clean code often performs better).

## Mental model / Intuition
Think of code quality as **friction in a machine**:
- High-quality code: smooth gears, easy to change, predictable behavior.
- Low-quality code: rusty gears, high friction, unpredictable side effects.

Small quality issues compound over time. A single 200-line function is annoying. A codebase with 500 such functions is unmaintainable.

## Decision rules (When to use / When not to use)
### Use it when
- Code will be maintained long-term (production systems, shared libraries).
- Multiple engineers will work on the codebase (quality enables collaboration).
- You need to onboard new engineers quickly.

### Avoid it when
- Writing throwaway code (prototypes, one-off scripts).
- Quality investment exceeds the value of the code (weekend hackathon project).

## How I would use it (practical)
- **Context:** Maintaining a Go API service in a team of 5 engineers.
- **Steps:**
  1. **Automate linting:** Run `golangci-lint` in CI (catches unused variables, error handling gaps, style violations).
  2. **Set complexity limits:** Fail CI if cyclomatic complexity >10 for any function (forces breaking down complex logic).
  3. **Require test coverage:** Fail CI if coverage drops below 80% (ensures new code is tested).
  4. **Code reviews:** Require 1 approval before merge; use checklist (error handling, naming, tests, security).
  5. **Refactor regularly:** Dedicate 10-20% of sprint capacity to tech debt (simplify complex functions, remove duplication).
- **What success looks like:** PR reviews take <30 minutes; no production bugs from missed error handling; new engineers productive in <2 weeks.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Reduces bugs (static analysis catches errors humans miss).
  - Faster development over time (easy to change clean code).
  - Lower cognitive load (readable code = less mental effort).
- **Cons / Risks:**
  - Upfront time cost (refactoring, writing tests, code reviews).
  - Can slow initial delivery if over-applied (perfect code for throwaway prototypes).
  - Requires discipline and tooling (CI setup, team buy-in).

### Alternatives
- **"Ship fast, clean up later":** Works for MVPs but creates tech debt that's rarely paid down.
- **Manual code review only:** Misses issues that tools catch automatically (unused variables, SQL injection risks).
- **No quality standards:** Fast initially but velocity drops as codebase grows (high defect rate, hard to change).

### How to choose
- **Long-lived production system:** Invest in full code quality (linters, reviews, tests, complexity limits).
- **Prototype / MVP:** Lightweight quality (basic linting, no coverage mandates).
- **Shared library:** Extra scrutiny (API design, documentation, backward compatibility).

## Failure modes & Pitfalls
- **Vanity metrics:** High test coverage but tests don't assert behavior (just call code without validation).
- **Overly strict rules:** Blocking PRs for minor style issues kills velocity and morale.
- **No refactoring budget:** Quality degrades over time; tech debt compounds.
- **"Code review as gatekeeper":** Reviewers nitpick style instead of focusing on correctness and design.
- **Ignoring complexity:** 500-line functions that no one understands; impossible to test or debug.

## Observability (How to detect issues)
- **Metrics:**
  - Cyclomatic complexity (per function; flag >10).
  - Test coverage (% of code executed by tests).
  - Linter violations (should be zero in main branch).
  - PR review time (long reviews suggest unclear code).
  - Defect rate (bugs per 1000 lines of code).
- **Logs:** Track linter violations over time (should decrease, not increase).
- **Traces:** N/A for code quality itself.
- **Alerts:** Complexity spike in new code, coverage drop below threshold.

## Implementation notes
- **Checklist**
  - [ ] Set up linter in CI (golangci-lint, ESLint, pylint, etc.).
  - [ ] Define complexity limits (cyclomatic complexity <10, function length <50 lines).
  - [ ] Require test coverage (>80% for new code, track trend).
  - [ ] Establish code review checklist (error handling, tests, naming, security).
  - [ ] Block merge on linter failures or test failures.
  - [ ] Dedicate time for refactoring (10-20% of sprint capacity).

- **Security / Compliance notes**
  - Include security linters (gosec, Bandit, semgrep) in CI.
  - Flag common vulnerabilities (SQL injection, XSS, secrets in code).

- **Performance notes**
  - Use profilers to identify hot paths (don't prematurely optimize).
  - Complexity often correlates with poor performance (simpler code is easier to optimize).

- **Operational notes**
  - Quality doesn't end at deploy: monitor production errors and trace them back to code issues.
  - Use post-mortems to identify code quality gaps (e.g., missing error handling, unclear logic).

## Mini example
```go
// BAD: High complexity, hard to test, unclear logic
func ProcessOrder(order Order) error {
    if order.Total > 0 {
        if order.UserID != "" {
            user, err := getUser(order.UserID)
            if err != nil {
                return err
            }
            if user.Status == "active" {
                if user.Balance >= order.Total {
                    // ... 50 more lines of nested logic
                    return nil
                } else {
                    return errors.New("insufficient balance")
                }
            } else {
                return errors.New("user not active")
            }
        } else {
            return errors.New("missing user ID")
        }
    } else {
        return errors.New("invalid order total")
    }
}

// GOOD: Low complexity, testable, clear logic
func ProcessOrder(order Order) error {
    if err := validateOrder(order); err != nil {
        return fmt.Errorf("invalid order: %w", err)
    }
    
    user, err := getActiveUser(order.UserID)
    if err != nil {
        return fmt.Errorf("user lookup failed: %w", err)
    }
    
    if user.Balance < order.Total {
        return ErrInsufficientBalance
    }
    
    return chargeUser(user, order.Total)
}

func validateOrder(order Order) error {
    if order.Total <= 0 {
        return errors.New("order total must be positive")
    }
    if order.UserID == "" {
        return errors.New("user ID is required")
    }
    return nil
}

func getActiveUser(userID string) (*User, error) {
    user, err := getUser(userID)
    if err != nil {
        return nil, err
    }
    if user.Status != "active" {
        return nil, ErrUserNotActive
    }
    return user, nil
}
```

## Common anti-patterns
- **Anti-pattern:** God objects / God functions (single function/class doing everything).
  - **Why it's bad:** Impossible to test, debug, or reason about; violates single responsibility principle.
  - **Better approach:** Break into smaller, focused functions/classes with clear responsibilities.

- **Anti-pattern:** Magic numbers and unclear names (`if status == 3`).
  - **Why it's bad:** Readers don't know what `3` means; requires digging through code or documentation.
  - **Better approach:** Use constants or enums (`if status == StatusActive`).

- **Anti-pattern:** Copy-paste code (duplication across multiple files).
  - **Why it's bad:** Bugs must be fixed in multiple places; high maintenance burden.
  - **Better approach:** Extract shared logic into functions or libraries (DRY: Don't Repeat Yourself).

- **Anti-pattern:** No error handling ("hope it works").
  - **Why it's bad:** Silent failures lead to production incidents.
  - **Better approach:** Check every error, log context, return or handle appropriately.

## Interview readiness
### "Explain it like I'm teaching"
Code quality is about making code easy to read, maintain, and change. We measure it with automated tools like linters (catch style and correctness issues), complexity metrics (flag overly complex functions), and test coverage (ensure code is tested). But quality isn't just metrics—it's also about clear naming, good structure, and thoughtful design. The goal is to balance quality with speed: invest enough to keep the codebase maintainable without over-engineering. High-quality code pays back over time because teams move faster and encounter fewer bugs.

### Trap questions (with answers)
1) **Q:** Doesn't focusing on code quality slow down delivery?
   - **A:** Initially yes, but it accelerates delivery long-term. Low-quality code creates tech debt that requires constant firefighting, risky changes, and slow debugging. Clean code is easy to change, so teams move faster after the initial investment.

2) **Q:** If we have 90% test coverage, is our code high quality?
   - **A:** Not necessarily. Coverage measures what's executed, not what's validated. You can have 90% coverage with tests that don't assert anything. Quality also includes readability, complexity, and design—not just tests.

3) **Q:** What's the most important code quality metric?
   - **A:** No single metric. Use a combination: cyclomatic complexity (simplicity), test coverage (confidence), linter violations (correctness), and code review feedback (design). Metrics are indicators, not goals.

4) **Q:** How do you balance refactoring with feature delivery?
   - **A:** Dedicate 10-20% of sprint capacity to tech debt. Refactor opportunistically (when touching code for features) and strategically (dedicate sprints to high-impact areas). Use metrics (complexity, defect rate) to prioritize.

5) **Q:** Should all code follow the same quality standards?
   - **A:** No, use risk-based standards. Production APIs need high quality (linters, tests, reviews). Throwaway prototypes don't. Shared libraries need extra scrutiny (API design, docs, backward compatibility).

### Quick self-check (5 items)
- [ ] I can name at least 3 code quality metrics and their purpose.
- [ ] I can explain why cyclomatic complexity matters and what it measures.
- [ ] I can describe how to balance code quality with delivery speed.
- [ ] I can give an example of a code smell and how to fix it.
- [ ] I can articulate the difference between test coverage and test quality.

## Links (NO duplication)
### Prerequisites
- [Software Quality Assurance](software-quality-assurance.md)
- (TODO) SOLID principles
- (TODO) Design patterns

### Related topics
- [Testing pyramid](testing-pyramid.md)
- [Quality gates](quality-gates.md)
- [Clean code](clean-code.md)

### Compare with
- [Testing pyramid](testing-pyramid.md) — Testing validates behavior; code quality measures structure and maintainability.

## Notes / Inbox
- Add examples from real projects: simplifying complex functions in Heimdall, refactoring Opportunity Actions for testability.
- Consider adding section on code review best practices (what to focus on, how to give feedback).
