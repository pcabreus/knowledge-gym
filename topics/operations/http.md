---
id: http
title: "HTTP (Hypertext Transfer Protocol)"
type: technology
status: learning
importance: 55
difficulty: medium
tags: [operations, networking, application]
canonical: true
aliases: []
created_at: 2026-01-21
updated_at: 2026-01-21
---

# HTTP (Hypertext Transfer Protocol)

## TL;DR (BLUF)
- HTTP is an application-layer protocol for request/response communication.
- Use it for APIs and web communication.
- Trade-off: request/response latency and statelessness constraints.

## Definition
**What it is:** A stateless application-layer protocol that defines methods, headers, status codes, and payload semantics for request/response interactions, usually over [TCP](tcp.md) and often secured with [TLS](tls.md).  
**Key terms:** request/response, status code, method, header, stateless, idempotency.

## Why it matters
- It underpins most APIs and web services.
- HTTP behavior shapes performance (latency, caching) and reliability.

## Scope & Non-goals
**In scope:** HTTP semantics, status codes, and operational trade-offs.  
**Out of scope / NOT solved by this:** API design decisions or authentication mechanisms.

## Mental model / Intuition
- HTTP is like a form submission: you send a request, you get a response with status and data.

## Decision rules (When to use / When not to use)
### Use it when
- You need standardized request/response semantics.
- You want broad compatibility across systems.
### Avoid it when
- You need streaming or bidirectional real-time communication.
- You need ultra-low latency with custom protocols.

## How I would use it (practical)
- **Context:** Public REST API for a product catalog.
- **Steps:**
  1) Define resources and methods.
  2) Choose status codes and error contracts.
  3) Add caching and timeouts.
- **What success looks like:** predictable responses and clear error handling.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** ubiquitous, well-understood, easy to debug.
- **Cons / Risks:** request/response overhead and latency.
### Alternatives
- **gRPC:** better for strict contracts and performance.
- **WebSockets:** better for real-time bidirectional streams.
- **How to choose:** use HTTP for broad interoperability; pick alternatives for specialized needs.

## Failure modes & Pitfalls
- Overloading 200 OK with error payloads.
- Missing timeouts causing stuck requests.
- Treating retries as safe for non-idempotent methods.

## Observability (How to detect issues)
- **Metrics:** latency p95/p99, error rate by status code, throughput.
- **Logs:** request IDs, status codes, error payloads.
- **Traces:** request spans with downstream timing.
- **Alerts:** spikes in 5xx errors or latency regressions.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Use correct status codes
  - [ ] Set timeouts and retries appropriately
- **Security / Compliance notes**
  - Use [TLS](tls.md) for any user or sensitive data.
- **Performance notes**
  - Enable compression and caching where safe.
- **Operational notes**
  - Define clear SLIs/SLOs for latency and error rates.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using HTTP for internal RPC without timeouts.
  - **Why it’s bad:** tail latency and resource leaks.
  - **Better approach:** set timeouts and consider gRPC if appropriate.

## Interview readiness
### “Explain it like I’m teaching”
- HTTP is a stateless request/response protocol used by the web. It defines methods and status codes so clients and servers can communicate reliably.

### Trap questions (with answers)
1) **Q:** Is HTTP always over TCP?
  - **A:** Mostly, but HTTP/3 runs over QUIC ([UDP](udp.md)-based).
2) **Q:** Is GET always safe to retry?
   - **A:** Only if it’s idempotent and the server respects that.
3) **Q:** Is HTTPS a different protocol?
   - **A:** It’s HTTP over [TLS](tls.md).

### Quick self-check (5 items)
- [ ] I can define HTTP clearly.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Networking basics](networking-basics.md)
- [TCP](tcp.md)

### Related topics
- [TLS](tls.md)
- [API design basics](../system-design/api-design-basics.md)

### Compare with
- [API design basics](../system-design/api-design-basics.md) — protocol vs API semantics.

## Notes / Inbox (optional)
- N/A
