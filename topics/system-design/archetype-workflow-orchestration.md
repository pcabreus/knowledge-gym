---
id: archetype-workflow-orchestration
title: "Workflow Orchestration (Long-Running Processes)"
type: pattern
status: learning
importance: 90
difficulty: hard
tags: [system-design, saga, orchestration, choreography, state-machine, workflows]
canonical: true
aliases: [saga, orchestration, choreography, state-machine-workflows]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Workflow Orchestration (Long-Running Processes)

## TL;DR
- Coordinate multi-step operations across services (often with external dependencies).
- Key challenges: partial failures and compensation, retries without duplication, visibility into state.
- Solutions: Saga pattern, workflow engine / state machine, outbox + idempotent steps.

## Where it hurts (why it hurts)
1. **Partial failures and compensation**: Ticket issuance fails after payment → must refund or retry
   - **Solution**: Saga pattern with compensation steps (undo); explicit rollback logic
2. **Retries without duplication**: Retry step → double execution (double email, double charge)
   - **Solution**: Idempotent steps (dedupe per step); persisted state
3. **Visibility into state**: "Where is my order?" → no clear answer without state tracking
   - **Solution**: Workflow engine stores state; queryable; timeouts and alerts

## Decision rules
- **Use when**: Multi-step processes (travel booking, onboarding, payment flows), long-running (hours/days), external dependencies
- **Avoid when**: Simple 2-step flows (over-engineering), everything synchronous and fast

## Trade-offs
- **Orchestration (central coordinator)**: Easier to reason, visibility; single point of failure
- **Choreography (event-driven)**: Decoupled, resilient; harder to understand flow
- **Choose**: Complex flows / need visibility → orchestration; simple flows / high decoupling → choreography

## Explicit example
Travel booking: reserve seat → charge payment → issue ticket → send confirmation. If ticket issuance fails after payment, must refund (compensation) or retry safely (idempotency). Orchestrator tracks state; retries with timeouts; exposes "booking status" API.

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [Saga Pattern](../architecture/saga-pattern.md), [Idempotency](archetype-idempotency-dedup.md), [Outbox Pattern](../architecture/outbox-pattern.md)
