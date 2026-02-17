---
title: Versioning and Deprecations
---

# Versioning and Deprecations

## Definition
Versioning and deprecations are practices for managing changes to APIs, services, or software, allowing for parallel versions and planned removal of old functionality.

## Why it matters
They prevent breaking changes for consumers, enable safe evolution, and provide clear communication about end-of-life timelines.

## When to use / when not to use
Use for any public or widely used API/service. Not needed for purely internal, short-lived prototypes.

## Trade-offs
- + Enables safe evolution and migration
- + Reduces risk of breaking consumers
- - Adds maintenance overhead
- - Can lead to technical debt if old versions are not removed

## Prerequisites
- [API design basics](../system-design/api-design-basics.md)

## Related topics
- [Contract Testing](../quality/contract-testing.md)
- [Integration Testing](../quality/integration-testing.md)

## Trap questions
1. Why is versioning important for APIs?
   - To avoid breaking existing consumers when making changes.
2. What is a risk of not deprecating old versions?
   - Accumulation of technical debt and security risks.
3. What is a best practice for deprecating an API?
   - Communicate timelines and provide migration paths.
