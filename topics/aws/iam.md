---
title: IAM (AWS)
---

# IAM (AWS)

## Definition
IAM (Identity and Access Management) is an AWS service for managing access to AWS resources securely using users, groups, roles, and policies.

## Why it matters
It enforces least-privilege access, secures sensitive operations, and enables compliance with security standards.

## When to use / when not to use
Use for all AWS environments to control access. Not needed for non-AWS resources.

## Trade-offs
- + Fine-grained access control
- + Auditable and manageable
- - Can be complex to configure
- - Overly broad permissions are a risk

## Prerequisites
- [Service Level Agreement (SLA)](../operations/service-level-agreement.md)

## Related topics
- [Data-use Controls](../operations/data-use-controls.md)
- [Lambda](lambda.md)

## Trap questions
1. What is IAM in AWS?
   - Identity and Access Management for controlling access to AWS resources.
2. What is a common pitfall with IAM?
   - Using wildcards or overly broad permissions.
3. When should you avoid IAM?
   - Only for non-AWS resources; always use for AWS.
