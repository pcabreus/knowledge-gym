---
title: AppSync
---

# AppSync

## Definition
AppSync is an AWS managed service for building GraphQL APIs with real-time data, authorization, and integration with other AWS services.

## Why it matters
It simplifies the creation of scalable, secure GraphQL APIs and supports real-time subscriptions, offline access, and data aggregation.

## When to use / when not to use
Use for building modern APIs that require flexible queries, real-time updates, or integration with AWS data sources. Not needed for simple REST APIs or when GraphQL is not required.

## Trade-offs
- + Managed, scalable GraphQL API
- + Real-time and offline support
- - Complexity in resolver logic
- - Cost and learning curve

## Prerequisites
- [GraphQL](../system-design/graphql.md)

## Related topics
- [DynamoDB](../databases/dynamodb.md)
- [API Gateway](../system-design/api-gateway.md)

## Trap questions
1. What is AppSync?
   - AWS managed GraphQL API service.
2. What is a common pitfall with AppSync?
   - Complex resolver logic and cost management.
3. When should you avoid AppSync?
   - For simple APIs or when GraphQL is not needed.
