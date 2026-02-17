---
title: Contract Testing
---

# Contract Testing

## Definition
Contract testing is a technique for verifying that two separate systems (such as a service and its consumer) can communicate and exchange data as expected, based on a shared contract (API schema, message format, etc.).

## Why it matters
It prevents integration failures due to mismatched expectations between services, especially in microservices and event-driven architectures.

## When to use / when not to use
Use when you have independently deployed services or teams, or when API/schema changes are frequent. Not needed for monoliths or tightly coupled systems.

## Trade-offs
- + Early detection of breaking changes
- + Enables independent deployments
- - Requires maintenance of contracts and test infrastructure
- - May not catch all integration issues (e.g., auth, network)

## Prerequisites
- [API design basics](../system-design/api-design-basics.md)
- [Testing fundamentals](../quality/testing-fundamentals.md)

## Related topics
- [Integration tests](../quality/testing-fundamentals.md)
- [Versioning APIs and Events](../system-design/versioning-apis-and-events.md)

## Trap questions
1. What is the main difference between contract testing and integration testing?
   - Contract testing checks the agreement between services; integration testing checks the full system working together.
2. What happens if you skip contract tests in a microservices environment?
   - You risk breaking consumers or producers when APIs change.
3. Can contract tests replace end-to-end tests?
   - No, they complement but do not replace full E2E tests.
