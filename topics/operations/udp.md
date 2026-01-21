---
id: udp
title: "UDP (User Datagram Protocol)"
type: technology
status: learning
importance: 45
difficulty: medium
tags: [operations, networking, transport]
canonical: true
aliases: []
created_at: 2026-01-21
updated_at: 2026-01-21
---

# UDP (User Datagram Protocol)

## TL;DR (BLUF)
- UDP is a connectionless, best-effort transport protocol with no ordering or reliability guarantees.
- Use it for latency-sensitive or loss-tolerant workloads.
- Trade-off: you must handle reliability at the application layer if needed.

## Definition
**What it is:** A transport-layer protocol that sends independent datagrams without connection setup, delivery guarantees, or ordering.  
**Key terms:** datagram, connectionless, best-effort, loss, reordering.

## Why it matters
- UDP enables low latency and simple transport for real-time systems.
- It shifts responsibility for reliability and ordering to the application.

## Scope & Non-goals
**In scope:** UDP characteristics, typical use cases, and trade-offs.  
**Out of scope / NOT solved by this:** encryption, congestion control, or application-level reliability.

## Mental model / Intuition
- Think of UDP as postcards: you send them and hope they arrive in order.

## Decision rules (When to use / When not to use)
### Use it when
- Latency is more important than perfect delivery.
- The application can tolerate loss or implement its own reliability.
### Avoid it when
- You require ordered, reliable delivery.
- You can’t handle loss at the application layer.

## How I would use it (practical)
- **Context:** Metrics collection, streaming media, or online gaming.
- **Steps:**
  1) Use UDP for small, frequent messages.
  2) Implement retries or sequencing only if required.
  3) Monitor loss and latency.
- **What success looks like:** low latency with acceptable loss rates.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** low overhead, low latency, no connection setup.
- **Cons / Risks:** loss, duplication, and reordering.
### Alternatives
- **[TCP](tcp.md):** reliable, ordered delivery for correctness-sensitive data.
- **How to choose:** use UDP when timeliness outweighs reliability.

## Failure modes & Pitfalls
- Silent packet loss without detection.
- Application logic assuming ordering that doesn’t exist.
- Fragmentation causing higher loss rates.

## Observability (How to detect issues)
- **Metrics:** packet loss rate, jitter, latency.
- **Logs:** dropped packet counters if available.
- **Traces:** gaps or out-of-order sequence numbers.
- **Alerts:** sustained loss or jitter spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Keep datagrams small to avoid fragmentation
  - [ ] Add sequence numbers if ordering matters
- **Security / Compliance notes**
  - Validate payloads and rate-limit to prevent abuse.
- **Performance notes**
  - Batch sends where possible.
- **Operational notes**
  - Monitor network jitter and loss.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using UDP for data that requires guaranteed delivery.
  - **Why it’s bad:** lost packets corrupt business workflows.
  - **Better approach:** use [TCP](tcp.md) or add application-level reliability.

## Interview readiness
### “Explain it like I’m teaching”
- UDP sends datagrams without a connection and makes no guarantees about delivery or order. It’s fast and lightweight but requires the app to handle loss if needed.

### Trap questions (with answers)
1) **Q:** Does UDP guarantee delivery?
   - **A:** No; it is best-effort.
2) **Q:** Can UDP packets arrive out of order?
   - **A:** Yes; ordering is not guaranteed.
3) **Q:** Is UDP always insecure?
   - **A:** No; security depends on the application (e.g., DTLS or encryption).

### Quick self-check (5 items)
- [ ] I can define UDP precisely.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Networking basics](networking-basics.md)
- [Network layers (OSI & TCP/IP)](network-layers.md)

### Related topics
- [TCP](tcp.md)
- [API design basics](../system-design/api-design-basics.md)

### Compare with
- [TCP](tcp.md) — best-effort datagrams vs reliable ordered stream.

## Notes / Inbox (optional)
- N/A
