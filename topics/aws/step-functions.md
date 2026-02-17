---
title: Step Functions
---

# Step Functions

## Definition
Step Functions is an AWS service for orchestrating serverless workflows, allowing you to coordinate multiple AWS services into business-critical applications using state machines.

## Why it matters
It enables reliable, visual, and auditable orchestration of complex workflows with error handling, retries, and compensation logic.

## When to use / when not to use
Use for multi-step workflows, long-running processes, or when you need orchestration across AWS services. Not needed for simple, single-step tasks.

## Trade-offs
- + Visual workflow design and monitoring
- + Built-in error handling and retries
- - Can become complex for large workflows
- - Cost and state machine limits

## Prerequisites
- [AWS Lambda](lambda.md)

## Related topics
- [API Gateway](../system-design/api-gateway.md)
- [DynamoDB](../databases/dynamodb.md)

## Trap questions
1. What is a Step Function?
   - AWS service for orchestrating serverless workflows.
2. What is a common pitfall with Step Functions?
   - Mixing orchestration with core business logic.
3. When should you avoid Step Functions?
   - For simple, single-step tasks or when orchestration is not needed.
