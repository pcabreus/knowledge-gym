---
title: GraphQL
---

# GraphQL

## Definition
GraphQL is a query language and runtime for APIs that enables clients to request exactly the data they need, and nothing more, from a single endpoint.

## Why it matters
It allows for flexible, efficient data retrieval and can reduce over-fetching and under-fetching compared to REST APIs.

## When to use / when not to use
Use for APIs with complex, evolving data needs or multiple clients. Not ideal for simple, fixed data structures or when caching is critical.

## Trade-offs
- + Flexible, efficient queries
- + Strongly typed schema
- - Complexity in resolver logic
- - Caching and performance challenges

## Prerequisites
- [API design basics](../system-design/api-design-basics.md)

## Related topics
- [AppSync](../operations/appsync.md)
- [REST vs GraphQL](rest-vs-graphql.md) (TODO if missing)

## Trap questions
1. What is GraphQL?
   - A query language and runtime for APIs.
2. What is a common pitfall with GraphQL?
   - N+1 query problems and complex resolver logic.
3. When should you avoid GraphQL?
   - For simple APIs or when caching is a top priority.
