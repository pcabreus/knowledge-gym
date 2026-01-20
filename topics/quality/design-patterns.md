---
id: design-patterns
title: "Design Patterns (Overview)"
type: concept
status: learning
importance: 75
difficulty: medium
tags: [design, architecture, oop, reusability, best-practices]
canonical: true
aliases: [gof-patterns, software-patterns]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Design Patterns (Overview)

## TL;DR (BLUF)
- Design patterns are reusable solutions to common software design problems (e.g., Strategy, Factory, Observer).
- Use them to solve recurring problems with proven approaches, not as blueprints to apply everywhere.
- Key trade-off: flexibility and reusability vs. simplicity and directness.

## Definition
**What it is:** Formalized best practices for solving common design problems in software, categorized into creational, structural, and behavioral patterns.  
**Key terms:** Gang of Four (GoF), creational patterns (how objects are created), structural patterns (how objects are composed), behavioral patterns (how objects interact), anti-pattern.

## Why it matters
- **Shared vocabulary:** "Use a Factory" conveys a complex idea quickly to other engineers.
- **Proven solutions:** Patterns codify solutions that have worked across many projects.
- **Easier maintenance:** Patterns create recognizable structure, making code easier to understand.
- **Interview relevance:** Design patterns are common in system design and architecture discussions.

## Scope & Non-goals
**In scope:**
- Overview of pattern categories (creational, structural, behavioral).
- When to use patterns and when they're overkill.
- Common patterns: Singleton, Factory, Strategy, Observer, Adapter, Decorator.

**Out of scope / NOT solved by this:**
- Exhaustive catalog of all patterns (23 GoF patterns + more).
- Implementation in specific languages (though examples provided in Go).
- Domain-specific patterns (microservices patterns, cloud patterns—separate topics).

## Mental model / Intuition
Think of design patterns as **architectural blueprints**:
- You don't design a house from scratch every time; you adapt proven layouts (open floor plan, split-level, etc.).
- Patterns are templates, not rigid rules—adapt them to your context.
- Using a pattern where it doesn't fit is like building a mansion for a dog (over-engineering).

## Decision rules (When to use / When not to use)
### Use it when
- You encounter a recurring design problem (e.g., "How do I create objects without coupling to specific classes?").
- Code is growing complex and hard to extend (patterns provide structure).
- Team benefits from shared vocabulary (patterns communicate design intent).

### Avoid it when
- Problem is simple and doesn't recur (direct solution is clearer).
- Applying patterns prematurely (YAGNI—You Aren't Gonna Need It).
- Team is unfamiliar with patterns (learning curve may slow delivery).

## How I would use it (practical)
- **Context:** Building a notification system that sends emails, SMS, and push notifications.
- **Steps:**
  1. **Identify problem:** Need to select notification method at runtime without hardcoding.
  2. **Choose pattern:** Strategy pattern (encapsulate algorithms, make them interchangeable).
  3. **Implementation:**
     - Define `Notifier` interface.
     - Create `EmailNotifier`, `SMSNotifier`, `PushNotifier` implementations.
     - `NotificationService` depends on `Notifier` interface, not concrete classes.
  4. **Benefit:** Adding Slack notifications requires only a new `SlackNotifier` class, no changes to existing code.
- **What success looks like:** New notification methods added in <1 hour with zero changes to core service.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Solve common problems with proven approaches.
  - Improve code structure and flexibility.
  - Shared vocabulary accelerates communication.
- **Cons / Risks:**
  - Can over-complicate simple problems.
  - Premature pattern application (solving problems you don't have).
  - Patterns are language/paradigm-specific (GoF patterns are OOP-focused).

### Alternatives
- **No patterns:** Solve problems directly. Works for simple, stable code.
- **Functional patterns:** Composition, higher-order functions, immutability (different paradigm).
- **Pragmatic approach:** Learn patterns, apply when they fit, don't force them.

### How to choose
- **Recurring, complex problem:** Use a pattern.
- **Simple, one-off problem:** Direct solution is clearer.
- **Refactoring:** Introduce patterns when code becomes hard to maintain.

## Failure modes & Pitfalls
- **Pattern fever:** Applying patterns everywhere, even when simple code suffices.
- **Wrong pattern:** Using Factory when Strategy fits better (mismatched solution).
- **Overuse of Singleton:** Singleton is often an anti-pattern (global state, hard to test).
- **Cargo-cult patterns:** Using patterns without understanding why (following rules blindly).
- **Ignoring simplicity:** Patterns should simplify, not complicate.

## Observability (How to detect issues)
- **Metrics:** N/A for patterns themselves.
- **Logs:** N/A for patterns themselves.
- **Traces:** N/A for patterns themselves.
- **Alerts:** N/A for patterns themselves.

## Implementation notes
- **Checklist**
  - [ ] Identify the recurring problem or design challenge.
  - [ ] Choose a pattern that fits (don't force a pattern).
  - [ ] Implement the pattern, adapting to your context.
  - [ ] Validate that the pattern simplifies code (not complicates it).

- **Security / Compliance notes**
  - Patterns don't inherently improve security, but structure can help (e.g., Adapter isolates third-party dependencies).

- **Performance notes**
  - Patterns may introduce slight overhead (indirection, interfaces), but usually negligible.
  - Don't sacrifice performance for pattern purity without profiling.

- **Operational notes**
  - Patterns make code easier to maintain and extend (when applied correctly).

## Common Patterns (Brief Overview)

### Creational Patterns (Object Creation)
- **Factory:** Create objects without specifying exact class. Use when object creation is complex or varies by context.
- **Builder:** Construct complex objects step-by-step. Use for objects with many optional parameters.
- **Singleton:** Ensure only one instance exists. Often an anti-pattern (global state, hard to test); use sparingly.

### Structural Patterns (Object Composition)
- **Adapter:** Wrap incompatible interface to match expected interface. Use when integrating third-party libraries.
- **Decorator:** Add behavior to objects dynamically without subclassing. Use for flexible feature combinations.
- **Facade:** Provide simplified interface to complex subsystem. Use to hide complexity from clients.

### Behavioral Patterns (Object Interaction)
- **Strategy:** Encapsulate algorithms, make them interchangeable. Use for runtime selection of behavior.
- **Observer:** Notify multiple objects when state changes. Use for event-driven systems.
- **Command:** Encapsulate requests as objects. Use for undo/redo, queueing, logging.

## Mini example (Strategy Pattern in Go)
```go
// Strategy pattern: encapsulate algorithms
type Notifier interface {
    Send(ctx context.Context, userID string, message string) error
}

type EmailNotifier struct {
    client *EmailClient
}

func (n *EmailNotifier) Send(ctx context.Context, userID string, message string) error {
    return n.client.SendEmail(userID, message)
}

type SMSNotifier struct {
    client *SMSClient
}

func (n *SMSNotifier) Send(ctx context.Context, userID string, message string) error {
    return n.client.SendSMS(userID, message)
}

// Context uses strategy
type NotificationService struct {
    notifier Notifier // Strategy injected
}

func (s *NotificationService) Notify(ctx context.Context, userID string, message string) error {
    return s.notifier.Send(ctx, userID, message) // Strategy executed
}

// Usage: select strategy at runtime
emailService := &NotificationService{notifier: &EmailNotifier{client: emailClient}}
smsService := &NotificationService{notifier: &SMSNotifier{client: smsClient}}
```

## Common anti-patterns
- **Anti-pattern:** Using Singleton for everything.
  - **Why it's bad:** Global state, hard to test, tight coupling.
  - **Better approach:** Use dependency injection; limit Singleton to truly global resources (logger, config).

- **Anti-pattern:** Applying patterns without understanding the problem.
  - **Why it's bad:** Over-complicates code; patterns should simplify, not obscure.
  - **Better approach:** Understand the problem first, then choose a pattern (if needed).

- **Anti-pattern:** God objects disguised as patterns.
  - **Why it's bad:** A "Manager" or "Handler" class doing everything isn't a pattern; it's a code smell.
  - **Better approach:** Follow SRP; break into focused classes.

## Interview readiness
### "Explain it like I'm teaching"
Design patterns are reusable solutions to common software design problems. They're categorized into creational (how to create objects), structural (how to compose objects), and behavioral (how objects interact). For example, the Strategy pattern lets you swap algorithms at runtime—instead of hardcoding "send email," you define a Notifier interface and create EmailNotifier, SMSNotifier, etc. Patterns provide a shared vocabulary (saying "use a Factory" is faster than explaining object creation logic) and proven solutions. The key is to use patterns when they fit, not force them everywhere—patterns should simplify code, not complicate it.

### Trap questions (with answers)
1) **Q:** Should you always use design patterns?
   - **A:** No. Use patterns when they solve a real problem. For simple, stable code, direct solutions are clearer. Patterns are tools, not mandates. Apply them when complexity justifies the structure.

2) **Q:** What's the most important design pattern?
   - **A:** No single pattern is most important; it depends on the problem. Strategy and Factory are very common for flexibility. Singleton is often overused (anti-pattern). Learn the problem each pattern solves, then apply appropriately.

3) **Q:** Aren't design patterns just for object-oriented programming?
   - **A:** GoF patterns are OOP-focused, but the principles apply to other paradigms. Functional programming has its own patterns (composition, higher-order functions, monads). The core idea—reusable solutions to common problems—transcends paradigms.

4) **Q:** How do you know which pattern to use?
   - **A:** Identify the problem first. Need to create objects without coupling? Factory. Need to swap algorithms at runtime? Strategy. Need to notify multiple objects of state changes? Observer. Don't start with a pattern; start with the problem.

5) **Q:** Can patterns hurt code quality?
   - **A:** Yes, if misapplied. Over-engineering with patterns makes code harder to understand (more indirection, more classes). Use patterns to simplify, not to demonstrate knowledge. If a direct solution is clearer, use it.

### Quick self-check (5 items)
- [ ] I can name at least 5 design patterns and categorize them (creational/structural/behavioral).
- [ ] I can explain when to use Strategy pattern and when it's overkill.
- [ ] I can describe why Singleton is often an anti-pattern.
- [ ] I can articulate the trade-off between flexibility (patterns) and simplicity (direct code).
- [ ] I can give an example of a problem and choose an appropriate pattern.

## Links (NO duplication)
### Prerequisites
- [SOLID principles](solid-principles.md)
- [Clean code](clean-code.md)

### Related topics
- [Code quality](code-quality.md)
- (TODO) Refactoring techniques
- (TODO) Strategy pattern (detailed)
- (TODO) Factory pattern (detailed)
- (TODO) Observer pattern (detailed)

### Compare with
- [SOLID principles](solid-principles.md) — SOLID provides design principles; patterns provide concrete solutions.

## Notes / Inbox
- Consider creating detailed topics for individual patterns (Strategy, Factory, Observer, etc.).
- Add examples from real projects: Factory for creating database clients, Strategy for notification routing.
- Link to "Refactoring to Patterns" concept when created.
