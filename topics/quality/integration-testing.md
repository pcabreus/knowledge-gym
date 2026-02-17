---
title: Integration Testing
---

# Integration Testing

## Definition
Integration testing verifies that multiple components or systems work together as intended, focusing on their interactions and data flow.

## Why it matters
It catches issues that unit tests miss, such as misconfigured dependencies, broken contracts, or real-world data problems.

## When to use / when not to use
Use when you have multiple components/services that interact. Not needed for isolated, pure functions.

## Trade-offs
- + Detects real integration issues
- + Increases confidence in system behavior
- - Slower and more complex than unit tests
- - Can be flaky if external dependencies are unstable

## Prerequisites
- [Testing fundamentals](../quality/testing-fundamentals.md)

## Related topics
- [Contract Testing](contract-testing.md)
- [Unit Testing](../quality/testing-fundamentals.md)

## Trap questions
1. Why are integration tests often slower than unit tests?
   - They involve real dependencies (DB, network, etc.), not just in-memory logic.
2. What is a common pitfall of integration testing?
   - Flaky tests due to unstable external systems.
3. Should you mock dependencies in integration tests?
   - No, use real or realistic dependencies to test actual integration.
