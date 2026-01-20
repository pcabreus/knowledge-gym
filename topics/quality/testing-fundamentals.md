---
id: testing-fundamentals
title: "Testing Fundamentals"
type: concept
status: learning
importance: 90
difficulty: easy
tags: [testing, quality, automation, verification]
canonical: true
aliases: [automated-testing, test-basics]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Testing Fundamentals

## TL;DR (BLUF)
- Testing validates that software behaves correctly under various conditions, catching bugs before production.
- Use automated tests for fast feedback and manual testing for exploratory scenarios.
- Key trade-off: time spent writing tests vs. risk of shipping defects.

## Definition
**What it is:** The practice of executing code under controlled conditions to verify it behaves as expected and to identify defects before they reach users.  
**Key terms:** test case, assertion, test fixture, test double (mock/stub/fake), code under test (CUT), system under test (SUT), regression testing, happy path, edge case.

## Why it matters
- **Prevents defects:** Catches bugs before they reach production (10-100x cheaper than fixing in production).
- **Enables refactoring:** Tests act as safety nets, allowing confident code changes.
- **Documents behavior:** Tests serve as executable specifications of how code should work.
- **Faster feedback:** Automated tests run in seconds/minutes vs. hours/days for manual QA.
- **Interview relevance:** Testing strategy is fundamental for senior engineering discussions.

## Scope & Non-goals
**In scope:**
- Test types (unit, integration, E2E, performance, etc.).
- Test anatomy (arrange, act, assert).
- Test doubles (mocks, stubs, fakes).
- When to test and what to test.

**Out of scope / NOT solved by this:**
- Specific testing frameworks (pytest, Jest, JUnit—those are tools).
- Quality assurance processes (broader than testing alone).

## Mental model / Intuition
Think of testing as **preventive medicine**:
- **Unit tests:** Daily vitamins (cheap, frequent, preventive).
- **Integration tests:** Regular checkups (catch issues between systems).
- **E2E tests:** Major diagnostics (expensive, comprehensive, less frequent).
- **Manual testing:** Specialist consultation (exploratory, creative, not routine).

You don't skip vitamins because you get annual checkups. Each layer serves a purpose.

## Decision rules (When to use / When not to use)
### Use it when
- Code will be maintained long-term (production systems).
- Multiple engineers work on the codebase (tests prevent regressions).
- Refactoring is necessary (tests enable safe changes).
- Bugs are costly (user impact, revenue loss, data corruption).

### Avoid it when
- Writing throwaway code (prototypes, one-off scripts).
- Test cost exceeds code value (weekend hackathon project).
- Learning a new technology (experiment first, test later).

## How I would use it (practical)
- **Context:** Building a Go API endpoint for user registration.
- **Steps:**
  1. **Unit test:** Validate business logic in isolation.
     - Test: valid email formats accepted, invalid rejected.
     - Test: password hashing works correctly.
     - Test: duplicate email returns proper error.
  2. **Integration test:** Test with real database.
     - Test: user record inserted correctly.
     - Test: transaction rolls back on error.
  3. **E2E test:** Test full flow via HTTP.
     - Test: POST /register → 201 response → user can login.
  4. **Edge cases:** Empty inputs, SQL injection attempts, concurrent registrations.
- **What success looks like:** All tests pass in <10 seconds; production has zero registration bugs in first month.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Catches bugs early (cheaper to fix).
  - Enables confident refactoring.
  - Documents expected behavior.
  - Faster than manual testing.
- **Cons / Risks:**
  - Upfront time cost (writing tests).
  - Maintenance burden (tests need updates when code changes).
  - False confidence (bad tests pass but code is broken).

### Alternatives
- **Manual testing only:** Slow, expensive, misses edge cases, not repeatable.
- **No testing ("hope it works"):** Fast initially but high production defect rate.
- **Testing in production:** Some scenarios require it (observability, canary deploys), but shouldn't be the primary strategy.

### How to choose
- **Production systems:** Automated testing (unit + integration + critical E2E).
- **Prototypes:** Minimal or no testing (validate concept first).
- **Critical systems:** Extensive testing (financial, healthcare, safety-critical).

## Failure modes & Pitfalls
- **Testing the wrong thing:** Testing implementation details instead of behavior (tests break on refactoring).
- **Flaky tests:** Tests that randomly fail due to timing, race conditions, environmental issues (erodes trust).
- **No assertions:** Tests that execute code but don't verify outcomes ("smoke tests" that don't catch bugs).
- **Too many mocks:** Over-mocking hides integration bugs (code passes tests but fails in production).
- **Ignoring edge cases:** Testing only happy paths (bugs hide in error conditions, boundary values).

## Observability (How to detect issues)
- **Metrics:**
  - Test pass rate (should be >99%).
  - Test execution time (unit tests <10s, integration <5min).
  - Test flakiness rate (% of random failures; aim for <1%).
  - Code coverage (% of code executed by tests).
- **Logs:** Track test failures by category (unit vs integration vs E2E).
- **Traces:** N/A for testing itself.
- **Alerts:** Test suite duration spike, flakiness increase, coverage drop.

## Implementation notes
- **Checklist**
  - [ ] Write tests as you code (TDD or test-soon-after).
  - [ ] Follow AAA pattern: Arrange (setup), Act (execute), Assert (verify).
  - [ ] Test behavior, not implementation details.
  - [ ] Use descriptive test names (what's being tested, expected outcome).
  - [ ] Test happy path + edge cases + error conditions.
  - [ ] Keep tests fast and deterministic (no sleeps, no random data).
  - [ ] Run tests in CI (block merge on failures).

- **Security / Compliance notes**
  - Include security tests: input validation, SQL injection, XSS prevention.
  - Test auth/authz (unauthorized access blocked, permissions enforced).

- **Performance notes**
  - Parallelize test execution where possible.
  - Use test doubles to avoid slow dependencies (DB, network).

- **Operational notes**
  - Fix or remove flaky tests immediately (they erode trust).
  - Monitor test suite performance (slow tests kill productivity).

## Mini example
```go
// Example: Testing user registration logic
package user

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

// Unit test: validate email format
func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        {"valid email", "user@example.com", false},
        {"missing @", "userexample.com", true},
        {"missing domain", "user@", true},
        {"empty", "", true},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateEmail(tt.email)
            if tt.wantErr {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
            }
        })
    }
}

// Integration test: database interaction
func TestCreateUser_Integration(t *testing.T) {
    // Arrange: setup test database
    db := setupTestDB(t)
    defer db.Close()
    
    user := User{
        Email:    "test@example.com",
        Password: "hashed_password",
    }
    
    // Act: create user
    err := CreateUser(db, user)
    
    // Assert: verify success
    assert.NoError(t, err)
    
    // Verify user exists in DB
    var count int
    db.QueryRow("SELECT COUNT(*) FROM users WHERE email = $1", user.Email).Scan(&count)
    assert.Equal(t, 1, count)
}

// Test edge case: duplicate email
func TestCreateUser_DuplicateEmail(t *testing.T) {
    db := setupTestDB(t)
    defer db.Close()
    
    user := User{Email: "duplicate@example.com", Password: "pass"}
    
    // Create user first time
    err := CreateUser(db, user)
    assert.NoError(t, err)
    
    // Attempt duplicate
    err = CreateUser(db, user)
    assert.Error(t, err)
    assert.Equal(t, ErrDuplicateEmail, err)
}
```

## Common anti-patterns
- **Anti-pattern:** No tests ("we'll test it manually").
  - **Why it's bad:** Manual testing is slow, inconsistent, and misses edge cases. Doesn't scale.
  - **Better approach:** Automate tests for fast, repeatable validation.

- **Anti-pattern:** Testing implementation details (e.g., "function X calls function Y").
  - **Why it's bad:** Tests break on refactoring even when behavior is correct.
  - **Better approach:** Test public API behavior (inputs → outputs), not internal calls.

- **Anti-pattern:** One giant test that tests everything.
  - **Why it's bad:** Hard to debug (which assertion failed?), hard to maintain, slow.
  - **Better approach:** Many small, focused tests (one behavior per test).

- **Anti-pattern:** Tests without assertions.
  - **Why it's bad:** Code executes but doesn't verify correctness; bugs slip through.
  - **Better approach:** Always assert expected outcomes (return values, state changes, side effects).

## Interview readiness
### "Explain it like I'm teaching"
Testing is how we verify software works correctly before it reaches users. We write code that exercises our production code and checks the results match expectations. There are different levels: unit tests check individual functions in isolation, integration tests verify components work together, and E2E tests validate entire user flows. Tests catch bugs early (when they're cheap to fix), enable safe refactoring (tests act as safety nets), and document expected behavior. The key is balancing test coverage with speed—write enough tests to catch critical bugs, but keep the suite fast so you get quick feedback.

### Trap questions (with answers)
1) **Q:** Why not just test in production? Users will tell us if something breaks.
   - **A:** Testing in production means users experience bugs, which damages trust and may cause data loss or revenue impact. Fixing bugs in production is 10-100x more expensive than catching them in development. Use testing to prevent defects, not discover them in production.

2) **Q:** If we have 100% code coverage, are we bug-free?
   - **A:** No. Coverage measures what code is executed, not what's validated. You can have 100% coverage with tests that don't assert anything. Also, coverage doesn't test edge cases, race conditions, or integration issues. Coverage is a signal of what's not tested, not a guarantee of quality.

3) **Q:** Should every line of code have a test?
   - **A:** No. Use risk-based testing. Focus on business logic, error handling, and edge cases. Simple getters/setters or trivial code may not need tests. Aim for high coverage on critical paths, not 100% coverage everywhere.

4) **Q:** How do you test code that calls external APIs?
   - **A:** Use test doubles. In unit tests, mock the API to isolate your logic. In integration tests, use a fake API server or test environment. In E2E tests, use the real API (or a staging version). Each layer validates different concerns.

5) **Q:** What's the difference between mocks, stubs, and fakes?
   - **A:** 
     - **Stub:** Returns hardcoded values (simplest, for queries).
     - **Mock:** Records calls and verifies expectations (for behavior verification).
     - **Fake:** Working implementation with shortcuts (e.g., in-memory DB instead of real DB).
     - Use stubs for most cases; mocks when you need to verify interactions; fakes for complex dependencies.

### Quick self-check (5 items)
- [ ] I can explain the purpose of testing (prevent defects, enable refactoring, document behavior).
- [ ] I can describe the AAA pattern (Arrange, Act, Assert).
- [ ] I can differentiate between unit, integration, and E2E tests.
- [ ] I can explain why testing implementation details is an anti-pattern.
- [ ] I can name at least 3 types of test doubles and when to use each.

## Links (NO duplication)
### Prerequisites
- [Software Quality Assurance](software-quality-assurance.md)

### Related topics
- [Testing pyramid](testing-pyramid.md)
- [Code quality](code-quality.md)
- [Quality gates](quality-gates.md)
- [Clean code](clean-code.md)

### Compare with
- [Testing pyramid](testing-pyramid.md) — Testing fundamentals explain what tests are; the pyramid explains how to balance test types.

## Notes / Inbox
- Add examples from real projects: testing Heimdall Step Functions logic, Opportunity Actions third-party API calls.
- Consider adding section on TDD (Test-Driven Development) workflow.
- Link to specific testing frameworks (Go: testing, testify; Python: pytest) when those topics are created.
