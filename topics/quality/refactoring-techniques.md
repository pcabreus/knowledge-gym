---
id: refactoring-techniques
title: "Refactoring Techniques"
type: skill
status: learning
importance: 80
difficulty: medium
tags: [refactoring, clean-code, maintainability, technical-debt]
canonical: true
aliases: [code-refactoring, refactoring-basics]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Refactoring Techniques

## TL;DR (BLUF)
- Refactoring is restructuring code to improve design without changing behavior—safer with tests as safety nets.
- Use small, incremental changes (extract function, rename variable, remove duplication) rather than large rewrites.
- Key trade-off: time spent improving structure vs. shipping features.

## Definition
**What it is:** The process of improving code structure, readability, and maintainability without altering its external behavior.  
**Key terms:** code smell (symptom of poor design), technical debt, red-green-refactor (TDD cycle), extract method/function, inline, rename, remove duplication, simplify conditionals.

## Why it matters
- **Prevents rot:** Code degrades over time without refactoring; becomes unmaintainable.
- **Enables velocity:** Clean code is faster to change; refactoring pays back in delivery speed.
- **Reduces bugs:** Simpler code has fewer places for bugs to hide.
- **Safer changes:** Tests ensure refactoring doesn't break behavior.
- **Interview relevance:** Senior engineers must demonstrate ability to improve code quality while maintaining correctness.

## Scope & Non-goals
**In scope:**
- Common refactoring techniques (extract function, rename, remove duplication, etc.).
- When to refactor and when to leave code alone.
- Safe refactoring workflow (tests → small changes → verify).

**Out of scope / NOT solved by this:**
- Rewrites (replacing entire systems—different from refactoring).
- Performance optimization (separate concern, though refactoring can enable it).

## Mental model / Intuition
Think of refactoring as **tidying a workspace**:
- Small, frequent tidying (daily): Easy, low-risk, keeps space usable.
- Letting mess accumulate (months): Eventually requires big cleanup (risky, expensive).
- Refactoring is continuous tidying, not once-a-year overhaul.

Also, refactoring with tests is like **editing with a safety net**: tests catch you if you break behavior.

## Decision rules (When to use / When not to use)
### Use it when
- Code is hard to understand or change (long functions, deep nesting, duplication).
- Adding features requires touching messy code (refactor first, then add feature).
- Tests exist (refactoring without tests is risky).
- Technical debt is slowing delivery.

### Avoid it when
- Code works and rarely changes (stable, low-churn areas).
- No tests (refactoring without tests risks breaking behavior).
- Feature deadline is immediate (refactor after shipping; or refactor opportunistically).

## How I would use it (practical)
- **Context:** Refactoring a 300-line Go function that processes orders.
- **Steps:**
  1. **Ensure tests exist:** If not, write characterization tests (capture current behavior).
  2. **Identify code smells:** Long function, deep nesting, duplicated validation logic.
  3. **Extract functions:** Break into `validateOrder()`, `fetchUser()`, `chargePayment()`, `sendConfirmation()`.
  4. **Run tests after each extraction:** Verify behavior unchanged.
  5. **Rename variables:** `o` → `order`, `u` → `user`, `res` → `result`.
  6. **Remove duplication:** Extract repeated validation into `validateEmail()`.
  7. **Simplify conditionals:** Replace nested if-else with early returns.
  8. **Commit incrementally:** Small commits, each with tests passing.
- **What success looks like:** Function is <50 lines; logic is clear; tests still pass; new feature takes <1 hour to add.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Improves maintainability (easier to change over time).
  - Reduces bugs (simpler code, fewer hiding spots).
  - Accelerates delivery long-term (clean code = faster iteration).
- **Cons / Risks:**
  - Upfront time cost (refactoring takes time).
  - Risk of breaking behavior (mitigated by tests).
  - Can be tempting to over-refactor (premature abstraction).

### Alternatives
- **Rewrite from scratch:** Risky; often takes longer than expected; may introduce new bugs.
- **Leave code as-is:** Fast short-term but accumulates debt (eventually slows delivery).
- **Opportunistic refactoring:** Refactor only when touching code for features (balanced approach).

### How to choose
- **Messy code with good tests:** Refactor incrementally.
- **Messy code without tests:** Add tests first, then refactor.
- **Stable, working code:** Leave it alone (don't fix what isn't broken).

## Failure modes & Pitfalls
- **Refactoring without tests:** High risk of breaking behavior; no safety net.
- **Big-bang refactoring:** Large, risky changes; hard to review and debug.
- **Premature abstraction:** Refactoring before you understand the problem (YAGNI violation).
- **Refactoring on a deadline:** Creates pressure to cut corners or skip tests.
- **Gold-plating:** Refactoring beyond what's needed (perfectionism).

## Observability (How to detect issues)
- **Metrics:**
  - Cyclomatic complexity (flag functions >10).
  - Function length (flag >50 lines).
  - Code churn (files changed frequently may need refactoring).
- **Logs:** N/A for refactoring itself.
- **Traces:** N/A for refactoring itself.
- **Alerts:** N/A for refactoring itself.

## Implementation notes
- **Checklist**
  - [ ] Ensure tests exist and pass (if not, write tests first).
  - [ ] Identify code smell (long function, duplication, deep nesting, unclear names).
  - [ ] Choose a refactoring technique (extract function, rename, simplify conditionals, etc.).
  - [ ] Make one small change at a time.
  - [ ] Run tests after each change (verify behavior unchanged).
  - [ ] Commit incrementally (small commits, each with tests passing).
  - [ ] Stop when code is "good enough" (don't over-refactor).

- **Security / Compliance notes**
  - Refactoring doesn't change behavior, so security should be unaffected.
  - Use tests to verify security constraints are preserved.

- **Performance notes**
  - Refactoring often improves performance (simpler code is easier to optimize).
  - If performance regresses, profile and adjust (but test first).

- **Operational notes**
  - Refactor in small PRs (easier to review, less risky).
  - Allocate 10-20% of sprint capacity to technical debt/refactoring.

## Common Refactoring Techniques

### Extract Function
**Smell:** Long function doing multiple things.  
**Technique:** Extract cohesive block into a new function with descriptive name.  
**Example:**
```go
// Before
func ProcessOrder(order Order) error {
    // 50 lines of validation
    // 50 lines of payment processing
    // 50 lines of notification
}

// After
func ProcessOrder(order Order) error {
    if err := validateOrder(order); err != nil {
        return err
    }
    if err := processPayment(order); err != nil {
        return err
    }
    return sendConfirmation(order)
}
```

### Rename Variable/Function
**Smell:** Unclear names (`x`, `tmp`, `proc()`).  
**Technique:** Rename to descriptive name.  
**Example:** `u` → `user`, `calc()` → `calculateDiscount()`.

### Remove Duplication (DRY)
**Smell:** Same logic repeated in multiple places.  
**Technique:** Extract shared logic into a function.  
**Example:**
```go
// Before
if user.Email == "" || !strings.Contains(user.Email, "@") {
    return errors.New("invalid email")
}
// ... 10 lines later
if admin.Email == "" || !strings.Contains(admin.Email, "@") {
    return errors.New("invalid email")
}

// After
func validateEmail(email string) error {
    if email == "" || !strings.Contains(email, "@") {
        return errors.New("invalid email")
    }
    return nil
}
```

### Simplify Conditionals
**Smell:** Deep nesting, complex if-else chains.  
**Technique:** Use early returns, guard clauses, or extract to functions.  
**Example:**
```go
// Before
func Charge(user User, amount int) error {
    if user.ID != "" {
        if user.Status == "active" {
            if user.Balance >= amount {
                // charge logic
                return nil
            } else {
                return ErrInsufficientBalance
            }
        } else {
            return ErrInactive
        }
    } else {
        return ErrInvalidUser
    }
}

// After
func Charge(user User, amount int) error {
    if user.ID == "" {
        return ErrInvalidUser
    }
    if user.Status != "active" {
        return ErrInactive
    }
    if user.Balance < amount {
        return ErrInsufficientBalance
    }
    // charge logic
    return nil
}
```

### Inline Function
**Smell:** Function that's only called once and doesn't add clarity.  
**Technique:** Inline the function body into the caller.  
**Use sparingly:** Only when function doesn't improve readability.

### Replace Magic Numbers with Constants
**Smell:** Hardcoded numbers (`if status == 3`).  
**Technique:** Define constants or enums.  
**Example:**
```go
// Before
if user.Status == 3 {
    // ...
}

// After
const (
    StatusInactive = 1
    StatusPending  = 2
    StatusActive   = 3
)
if user.Status == StatusActive {
    // ...
}
```

## Common anti-patterns
- **Anti-pattern:** Refactoring without tests.
  - **Why it's bad:** No safety net; high risk of breaking behavior.
  - **Better approach:** Write tests first (or at minimum, characterization tests).

- **Anti-pattern:** Big-bang refactoring (rewrite entire module).
  - **Why it's bad:** Risky, hard to review, blocks other work.
  - **Better approach:** Incremental refactoring (small changes, frequent commits).

- **Anti-pattern:** Refactoring for the sake of refactoring.
  - **Why it's bad:** Wastes time; code churn without benefit.
  - **Better approach:** Refactor when adding features or fixing bugs (opportunistic).

## Interview readiness
### "Explain it like I'm teaching"
Refactoring is improving code structure without changing behavior. It's like tidying a workspace—small, frequent cleanups keep things usable. Common techniques include extracting long functions into smaller ones, renaming unclear variables, removing duplication, and simplifying nested conditionals. The key is to refactor in small steps with tests as safety nets. Each change should be small enough to verify quickly. Refactoring prevents technical debt from accumulating and keeps code maintainable. The trade-off is time—refactoring takes time upfront but pays back in faster delivery long-term.

### Trap questions (with answers)
1) **Q:** When should you refactor?
   - **A:** Refactor opportunistically (when touching code for features), when code is hard to understand or change, or when technical debt is slowing delivery. Don't refactor code that works and rarely changes. Always ensure tests exist before refactoring.

2) **Q:** How do you know when to stop refactoring?
   - **A:** Stop when code is "good enough"—readable, testable, and easy to change. Don't pursue perfection (diminishing returns). Use heuristics: functions <50 lines, complexity <10, no obvious duplication. If adding a feature is easy, you're done.

3) **Q:** What if you don't have tests?
   - **A:** Write characterization tests first (tests that capture current behavior, even if it's buggy). Then refactor. Refactoring without tests is risky—you have no way to verify you didn't break anything.

4) **Q:** Isn't refactoring just wasted time? Why not rewrite?
   - **A:** Rewrites are risky: they take longer than expected, often introduce new bugs, and may not deliver value. Refactoring is incremental and safe (with tests). Refactor when the cost is justified; rewrite only when the system is fundamentally broken.

5) **Q:** How do you refactor legacy code with no tests?
   - **A:** Add tests incrementally around the area you need to change (characterization tests). Refactor small pieces at a time. Use techniques like "sprout method" (add new, tested function; call from legacy code) to gradually improve. It's slow but safer than big rewrites.

### Quick self-check (5 items)
- [ ] I can name at least 5 refactoring techniques (extract function, rename, remove duplication, simplify conditionals, etc.).
- [ ] I can explain why refactoring without tests is risky.
- [ ] I can describe the red-green-refactor cycle (TDD).
- [ ] I can identify a code smell (long function, deep nesting, duplication) and fix it.
- [ ] I can articulate when to refactor and when to leave code alone.

## Links (NO duplication)
### Prerequisites
- [Clean code](clean-code.md)
- [Testing fundamentals](testing-fundamentals.md)

### Related topics
- [Code quality](code-quality.md)
- [SOLID principles](solid-principles.md)
- [Design patterns](design-patterns.md)

### Compare with
- [Clean code](clean-code.md) — Clean code is the goal; refactoring is the process to get there.

## Notes / Inbox
- Add examples from real projects: refactoring Heimdall handlers, simplifying Opportunity Actions logic.
- Consider adding section on "Strangler Fig" pattern for refactoring large systems.
- Link to Martin Fowler's "Refactoring" book (external link in future versions).
