---
id: solid-principles
title: "SOLID Principles"
type: concept
status: learning
importance: 80
difficulty: medium
tags: [design, oop, architecture, maintainability, clean-code]
canonical: true
aliases: [solid, object-oriented-design]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# SOLID Principles

## TL;DR (BLUF)
- SOLID is five object-oriented design principles that promote maintainable, flexible code: Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion.
- Use them to guide class/module design, especially when code grows complex.
- Key trade-off: design simplicity vs. flexibility for future changes.

## Definition
**What it is:** A set of five design principles for object-oriented programming that help create code that is easier to maintain, extend, and test.  
**Key terms:**
- **S**ingle Responsibility Principle (SRP)
- **O**pen/Closed Principle (OCP)
- **L**iskov Substitution Principle (LSP)
- **I**nterface Segregation Principle (ISP)
- **D**ependency Inversion Principle (DIP)

## Why it matters
- **Maintainability:** SOLID code is easier to understand, modify, and debug.
- **Testability:** Loosely coupled code is easier to test in isolation.
- **Flexibility:** Code designed with SOLID is easier to extend without breaking existing functionality.
- **Interview relevance:** SOLID principles are fundamental for senior engineering design discussions.

## Scope & Non-goals
**In scope:**
- Understanding each SOLID principle with practical examples.
- When to apply SOLID and when it's overkill.
- Trade-offs between flexibility and simplicity.

**Out of scope / NOT solved by this:**
- Specific design patterns (Strategy, Factory, etc.—though SOLID enables them).
- Functional programming paradigms (SOLID is OOP-focused, though principles apply conceptually).

## Mental model / Intuition
Think of SOLID as **building blocks for flexible systems**:
- **SRP:** Each block does one thing (Lego brick, not Swiss Army knife).
- **OCP:** Can add new blocks without modifying existing ones (extension, not modification).
- **LSP:** Replacement blocks work the same way (substitutability).
- **ISP:** Only connect blocks that fit (small, focused interfaces).
- **DIP:** High-level blocks don't depend on low-level blocks (abstractions, not concretions).

## Decision rules (When to use / When not to use)
### Use it when
- Code is growing complex and hard to maintain.
- Multiple engineers work on the codebase (SOLID enables collaboration).
- Requirements change frequently (SOLID makes code flexible).
- Building long-lived production systems.

### Avoid it when
- Writing throwaway code (prototypes, one-off scripts).
- Code is simple and unlikely to change (SOLID can over-complicate).
- Team is unfamiliar with SOLID (learning curve may slow delivery initially).

## How I would use it (practical)
- **Context:** Refactoring a Go notification system that sends emails, SMS, and push notifications.
- **Steps:**
  1. **SRP:** Split monolithic `NotificationService` into `EmailSender`, `SMSSender`, `PushSender` (each does one thing).
  2. **OCP:** Define `Notifier` interface; new notification types (Slack) don't modify existing code.
  3. **LSP:** All `Notifier` implementations follow same contract (send message, return error).
  4. **ISP:** Don't force all notifiers to implement `GetDeliveryReport` if only email supports it; split interfaces.
  5. **DIP:** High-level `NotificationManager` depends on `Notifier` interface, not concrete `EmailSender`.
- **What success looks like:** Adding Slack notifications requires zero changes to existing code; only new `SlackNotifier` implementation.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Easier to extend (new features don't break existing code).
  - Easier to test (small, focused units).
  - Reduces coupling (changes localized).
- **Cons / Risks:**
  - More classes/interfaces (can feel over-engineered for simple cases).
  - Upfront design cost (thinking through abstractions).
  - Can lead to premature abstraction (YAGNI violation).

### Alternatives
- **No principles:** Fast initially but becomes unmaintainable as code grows.
- **Functional programming:** Different paradigm but shares goals (immutability, pure functions, composition).
- **Pragmatic OOP:** Apply SOLID selectively where complexity justifies it.

### How to choose
- **Complex, evolving systems:** Apply SOLID to manage complexity.
- **Simple, stable code:** Keep it simple; don't over-engineer.
- **Refactoring:** Introduce SOLID when code becomes hard to maintain.

## Failure modes & Pitfalls
- **Premature abstraction:** Creating interfaces before you need them (violates YAGNI).
- **Interface explosion:** Too many small interfaces; hard to navigate codebase.
- **Violating SRP:** God classes that do everything (impossible to test or maintain).
- **Misunderstanding LSP:** Subclasses that change behavior in surprising ways (breaks polymorphism).
- **Over-applying DIP:** Injecting dependencies everywhere (adds complexity without benefit).

## Observability (How to detect issues)
- **Metrics:**
  - Class size (lines of code; flag >500).
  - Coupling (how many dependencies a class has; flag >10).
  - Method complexity (cyclomatic complexity; flag >10).
- **Logs:** N/A for SOLID itself.
- **Traces:** N/A for SOLID itself.
- **Alerts:** N/A for SOLID itself.

## Implementation notes
- **Checklist**
  - [ ] Ensure each class has one reason to change (SRP).
  - [ ] Design for extension, not modification (OCP).
  - [ ] Verify subclasses can replace base classes (LSP).
  - [ ] Keep interfaces small and focused (ISP).
  - [ ] Depend on abstractions, not concretions (DIP).

- **Security / Compliance notes**
  - SOLID makes security easier (small, focused classes are easier to audit).

- **Performance notes**
  - SOLID may introduce slight overhead (interfaces, indirection), but usually negligible.
  - Premature optimization is the enemy; prioritize maintainability.

- **Operational notes**
  - SOLID code is easier to debug (clear responsibilities, loose coupling).

## Mini example (Go)
```go
// BEFORE: Violates SRP, OCP, DIP
type NotificationService struct {
    emailClient *EmailClient
    smsClient   *SMSClient
}

func (s *NotificationService) Send(userID string, message string, method string) error {
    if method == "email" {
        return s.emailClient.Send(userID, message)
    } else if method == "sms" {
        return s.smsClient.Send(userID, message)
    }
    return errors.New("unsupported method")
}
// Problem: Adding Slack requires modifying Send() (violates OCP)
// Problem: Depends on concrete EmailClient/SMSClient (violates DIP)

// AFTER: Follows SRP, OCP, DIP
type Notifier interface { // DIP: depend on abstraction
    Send(ctx context.Context, userID string, message string) error
}

type EmailNotifier struct {
    client *EmailClient
}

func (n *EmailNotifier) Send(ctx context.Context, userID string, message string) error {
    return n.client.Send(userID, message)
}

type SMSNotifier struct {
    client *SMSClient
}

func (n *SMSNotifier) Send(ctx context.Context, userID string, message string) error {
    return n.client.Send(userID, message)
}

type NotificationManager struct {
    notifiers map[string]Notifier // DIP: depend on interface
}

func (m *NotificationManager) Send(ctx context.Context, method string, userID string, message string) error {
    notifier, ok := m.notifiers[method]
    if !ok {
        return errors.New("unsupported method")
    }
    return notifier.Send(ctx, userID, message) // OCP: adding Slack doesn't modify this code
}
// SRP: Each notifier has single responsibility
// OCP: Add SlackNotifier without changing NotificationManager
// LSP: All notifiers follow same contract
// DIP: NotificationManager depends on Notifier interface
```

## Each Principle Explained

### Single Responsibility Principle (SRP)
**Definition:** A class should have one, and only one, reason to change.

**Example violation:** `UserService` handles authentication, authorization, notifications, and logging.  
**Fix:** Split into `AuthService`, `AuthzService`, `NotificationService`, `Logger`.

**Why it matters:** Small, focused classes are easier to understand, test, and modify.

### Open/Closed Principle (OCP)
**Definition:** Software entities should be open for extension but closed for modification.

**Example violation:** Adding a new payment method requires modifying `PaymentProcessor` class.  
**Fix:** Define `PaymentMethod` interface; add new implementations without changing existing code.

**Why it matters:** Extend functionality without risking existing code (fewer regressions).

### Liskov Substitution Principle (LSP)
**Definition:** Subclasses should be substitutable for their base classes without altering correctness.

**Example violation:** `Square` extends `Rectangle` but breaks `setWidth()`/`setHeight()` contract (changing width also changes height).  
**Fix:** Don't use inheritance where it doesn't fit; use composition or separate hierarchies.

**Why it matters:** Polymorphism works correctly only when substitutability holds.

### Interface Segregation Principle (ISP)
**Definition:** Clients should not be forced to depend on interfaces they don't use.

**Example violation:** `Worker` interface has `work()`, `eat()`, `sleep()`. `Robot` implements `Worker` but doesn't need `eat()` or `sleep()`.  
**Fix:** Split into `Workable`, `Eatable`, `Sleepable` interfaces.

**Why it matters:** Small interfaces reduce coupling and make implementations simpler.

### Dependency Inversion Principle (DIP)
**Definition:** High-level modules should not depend on low-level modules. Both should depend on abstractions.

**Example violation:** `OrderService` directly instantiates `MySQLDatabase`.  
**Fix:** `OrderService` depends on `Database` interface; inject implementation (MySQL, PostgreSQL, fake).

**Why it matters:** Loose coupling enables testing, flexibility, and swapping implementations.

## Common anti-patterns
- **Anti-pattern:** God class (one class doing everything).
  - **Why it's bad:** Violates SRP; impossible to test or maintain.
  - **Better approach:** Split into focused classes with single responsibilities.

- **Anti-pattern:** Modifying existing code to add features.
  - **Why it's bad:** Violates OCP; risks breaking existing functionality.
  - **Better approach:** Use interfaces and polymorphism to extend without modifying.

- **Anti-pattern:** Subclasses that change base class behavior unexpectedly.
  - **Why it's bad:** Violates LSP; breaks polymorphism and causes bugs.
  - **Better approach:** Ensure subclasses honor base class contracts.

## Interview readiness
### "Explain it like I'm teaching"
SOLID is five design principles for writing maintainable object-oriented code. SRP says each class should do one thing. OCP says you should extend code without modifying it (use interfaces). LSP says subclasses should work anywhere the base class works. ISP says keep interfaces small and focused. DIP says depend on abstractions, not concrete implementations. Together, these principles make code easier to test, extend, and maintain. They're especially valuable as systems grow complex and requirements change.

### Trap questions (with answers)
1) **Q:** Doesn't SOLID lead to too many classes and complexity?
   - **A:** It can, if over-applied. Use SOLID when complexity justifies it (evolving requirements, multiple engineers, long-lived systems). For simple, stable code, keep it simple. SOLID is a tool, not a mandate.

2) **Q:** What's the most important SOLID principle?
   - **A:** SRP (Single Responsibility). If each class does one thing, most other principles follow naturally. SRP is the foundation; the others build on it.

3) **Q:** Can you apply SOLID in non-OOP languages like Go?
   - **A:** Yes, conceptually. Go doesn't have classes but has structs and interfaces. SRP applies to functions/packages. OCP uses interfaces for extension. DIP uses interface injection. The principles adapt to the paradigm.

4) **Q:** How do you know when you're violating SRP?
   - **A:** Ask "how many reasons could this class change?" If the answer is more than one (e.g., changing business logic AND changing data storage), it violates SRP. Also watch for long classes (>500 lines) or vague names like `Manager`, `Helper`, `Util`.

5) **Q:** Doesn't DIP make code harder to follow (too much indirection)?
   - **A:** Sometimes, yes. Use DIP when you need flexibility (testing, swapping implementations, decoupling layers). For simple, stable dependencies, direct coupling is fine. Balance flexibility with simplicity.

### Quick self-check (5 items)
- [ ] I can name all five SOLID principles and explain each in one sentence.
- [ ] I can give an example of violating SRP and how to fix it.
- [ ] I can explain the difference between OCP and LSP.
- [ ] I can describe when SOLID is overkill (and when it's essential).
- [ ] I can refactor a God class into focused classes following SRP.

## Links (NO duplication)
### Prerequisites
- [Clean code](clean-code.md)
- [Code quality](code-quality.md)

### Related topics
- [Testing fundamentals](testing-fundamentals.md)
- [Design patterns](design-patterns.md)
- [Refactoring techniques](refactoring-techniques.md)

### Compare with
- [Clean code](clean-code.md) — Clean code focuses on readability; SOLID focuses on structure and flexibility.

## Notes / Inbox
- Add examples from real projects: refactoring Heimdall for testability, Opportunity Actions notification system.
- Consider adding section on SOLID in Go vs. other languages (differences in applying principles).
- Link to specific design patterns (Strategy, Factory, Observer) when created.
