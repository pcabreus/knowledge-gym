---
title: Third-party Data APIs
---

# Third-party Data APIs

## Definition
Third-party data APIs are external interfaces provided by vendors or platforms to access data or services, such as email, CRM, or analytics.

## Why it matters
They enable rapid integration of external capabilities, but introduce risks around reliability, privacy, and compliance.

## When to use / when not to use
Use when you need to enrich your application with external data or services. Not needed for purely internal systems.

## Trade-offs
- + Rapid access to external capabilities
- + Reduces development time
- - Vendor lock-in and changing APIs
- - Privacy, compliance, and reliability risks

## Prerequisites
- [Webhooks](webhooks.md)

## Related topics
- [Nylas](nylas.md)
- [Gong](gong.md)

## Trap questions
1. What is a third-party data API?
   - An external interface to access data or services from another provider.
2. What is a common risk with third-party APIs?
   - Breaking changes or outages outside your control.
3. How can you mitigate risks with third-party APIs?
   - Use retries, monitoring, and clear contracts.
