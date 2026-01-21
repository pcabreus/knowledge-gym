---
id: networking-basics
title: "Networking Basics"
type: concept
status: learning
importance: 40
difficulty: easy
tags: [operations, networking]
canonical: true
aliases: []
created_at: 2026-01-19
updated_at: 2026-01-19
---

# Networking Basics

## TL;DR (BLUF)
- Networking basics cover latency, connections, and timeouts.
- Use them to reason about client-server performance.
- Trade-off: tighter timeouts reduce tail latency but increase errors.

## Definition
**What it is:** Fundamental concepts of network communication.
**Key terms:** latency, bandwidth, timeout, retry.

## Why it matters
- Network behavior impacts database connectivity and performance.
- Misconfigured timeouts cause cascading failures.

## Scope & Non-goals
**In scope:** basic networking concepts.
**Out of scope / NOT solved by this:** deep TCP internals.

## Mental model / Intuition
- Networks are unreliable; design for timeouts and retries.

## Decision rules (When to use / When not to use)
### Use it when
- You troubleshoot connection issues.
### Avoid it when
- Issues are clearly local to the DB process.

## How I would use it (practical)
- **Context:** DB client timeouts.
- **Steps:** check latency → tune timeouts → monitor errors.
- **What success looks like:** stable connections and fewer timeouts.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** better resilience with correct timeouts.
- **Cons / Risks:** overly aggressive timeouts cause errors.
### Alternatives
- **Local caching:** reduce network calls.
- **How to choose:** tune timeouts based on observed latency.

## Failure modes & Pitfalls
- Retrying without backoff causing spikes.

## Observability (How to detect issues)
- **Metrics:** latency p95, error rate.
- **Logs:** connection errors.
- **Alerts:** spikes in timeout errors.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Set timeouts
  - [ ] Use exponential backoff

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Infinite retries.
  - **Why it’s bad:** traffic storms.
  - **Better approach:** bounded retries with backoff.

## Interview readiness
### “Explain it like I’m teaching”
- Networking basics are about latency, bandwidth, and timeouts. Correct timeout and retry settings are essential for stable services.

### Trap questions (with answers)
1) **Q:** Are timeouts optional?
   - **A:** no; they prevent resource leaks.
2) **Q:** Is retry always safe?
   - **A:** no; retries can amplify load.
3) **Q:** Can local caching replace networking concerns?
   - **A:** no; network calls still happen.

### Quick self-check (5 items)
- [ ] I can define latency and timeout.
- [ ] I can name a trade-off.
- [ ] I can describe a pitfall.
- [ ] I can identify a monitoring signal.
- [ ] I can link to DB clients.

## Links (NO duplication)
### Prerequisites
- [Reliability basics](reliability-basics.md)

### Related topics
- [Network layers (OSI & TCP/IP)](network-layers.md)
- [DNS](dns.md)
- [TCP](tcp.md)
- [UDP](udp.md)
- [TLS](tls.md)
- [HTTP](http.md)
- [Database client basics](../databases/database-client-basics.md)

### Compare with
- [Connection pooling](../databases/connection-pooling.md) — network vs DB limits.
