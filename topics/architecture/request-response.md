---
id: request-response
title: "Request-Response Architecture"
type: concept
status: learning
importance: 40
difficulty: easy
tags: [architecture, communication]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Request-Response Architecture

## TL;DR (BLUF)
- Request-response is synchronous communication between services.
- Use it for low-latency, immediate responses.
- Trade-off: tighter coupling and higher failure impact.

## Definition
**What it is:** A synchronous pattern where a client sends a request and waits for a response.
**Key terms:** synchronous, timeout, retry.

## Why it matters
- It’s simple and easy to reason about.
- It can propagate failures and increase latency.

## Scope & Non-goals
**In scope:** synchronous communication trade-offs.
**Out of scope / NOT solved by this:** async messaging patterns.

## Mental model / Intuition
- Like a phone call: you wait for a response.

## Decision rules (When to use / When not to use)
### Use it when
- You need immediate results.
### Avoid it when
- The work can be asynchronous.

## How I would use it (practical)
- **Context:** User profile lookup.
- **Steps:** request → response → handle timeouts.
- **What success looks like:** low latency with clear error handling.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** simplicity and immediate response.
- **Cons / Risks:** coupling and cascading failures.
### Alternatives
- **Messaging:** async processing.
- **How to choose:** use request-response for strict latency requirements.

## Failure modes & Pitfalls
- Cascading failures due to synchronous dependencies.

## Observability (How to detect issues)
- **Metrics:** latency, error rate.
- **Logs:** timeout errors.
- **Alerts:** elevated p95 latency.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Set timeouts
  - [ ] Use retries with backoff

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Long chains of synchronous calls.
  - **Why it’s bad:** high latency and failure propagation.
  - **Better approach:** use async where possible.

## Interview readiness
### “Explain it like I’m teaching”
- Request-response is a synchronous pattern where the client waits for a reply. It’s simple but can increase coupling and latency.

### Trap questions (with answers)
1) **Q:** Are retries always safe?
   - **A:** no; they can amplify load and duplicate work.
2) **Q:** Can request-response be fully reliable?
   - **A:** it depends on timeouts and error handling.
3) **Q:** Is async always better?
   - **A:** no; some use cases need immediate responses.

### Quick self-check (5 items)
- [ ] I can define request-response.
- [ ] I can state when to use it.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can explain observability signals.

## Links (NO duplication)
### Prerequisites
- N/A

### Related topics
- [Messaging basics](messaging-basics.md)

### Compare with
- [Event-driven basics](event-driven-basics.md) — async vs sync.
