---
title: Lambda
---

# Lambda

## Definition
AWS Lambda is a serverless compute service that runs code in response to events and automatically manages the underlying compute resources.

## Why it matters
It enables event-driven, scalable, and cost-effective execution of code without managing servers.

## When to use / when not to use
Use for event-driven workloads, APIs, or automation tasks. Not ideal for long-running or stateful applications.

## Trade-offs
- + No server management
- + Scales automatically
- - Cold starts and timeout limits
- - Limited execution time and resources

## Prerequisites
- [API Gateway](../system-design/api-gateway.md)

## Related topics
- [Step Functions](step-functions.md)
- [DynamoDB](../databases/dynamodb.md)

## Trap questions
1. What is AWS Lambda?
   - A serverless compute service that runs code in response to events.
2. What is a common pitfall with Lambda?
   - Cold starts and managing timeouts/memory.
3. When should you avoid Lambda?
   - For long-running or stateful workloads.
