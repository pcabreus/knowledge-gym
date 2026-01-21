---
id: tcp
title: "TCP (Transmission Control Protocol)"
type: technology
status: learning
importance: 50
difficulty: medium
tags: [operations, networking, transport]
canonical: true
aliases: []
created_at: 2026-01-21
updated_at: 2026-01-21
---

# TCP (Transmission Control Protocol)

## TL;DR (BLUF)
- TCP is a reliable, connection-oriented transport protocol with ordered delivery.
- Use it for correctness-sensitive data (APIs, databases, file transfer).
- Trade-off: higher overhead and latency vs UDP.

## Definition
**What it is:** A transport-layer protocol that provides reliable, ordered, byte-stream delivery between endpoints using acknowledgements and retransmissions.  
**Key terms:** connection, handshake, ACK, retransmission, congestion control, flow control.

## Why it matters
- Most application protocols ([HTTP](http.md), database drivers) assume TCP’s reliability.
- TCP behavior affects latency, throughput, and tail performance.

## Scope & Non-goals
**In scope:** TCP guarantees, handshake, flow/congestion control, and typical use cases.  
**Out of scope / NOT solved by this:** application-level retry semantics or encryption ([TLS](tls.md)).

## Mental model / Intuition
- Think of TCP as a tracked delivery service: you get receipts (ACKs) and re-send if lost.
- It optimizes for correctness and order, not minimal latency.

## Decision rules (When to use / When not to use)
### Use it when
- You need reliable, ordered delivery.
- The application can tolerate some latency for correctness.
### Avoid it when
- You need ultra-low latency and can handle loss or reordering.
- You are sending real-time audio/video where drops are preferable to delays.

## How I would use it (practical)
- **Context:** API calls between services and database connections.
- **Steps:**
  1) Use TCP-based protocols ([HTTP](http.md), gRPC, database drivers).
  2) Set connection and request timeouts.
  3) Observe retransmissions and latency spikes.
- **What success looks like:** stable connections, predictable latency, and low error rates.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** reliable, ordered delivery; widespread support.
- **Cons / Risks:** higher overhead, head-of-line blocking, and slower recovery on loss.
### Alternatives
- **[UDP](udp.md):** lower latency but no delivery guarantees.
- **How to choose:** default to TCP unless you can tolerate loss and need lower latency.

## Failure modes & Pitfalls
- Head-of-line blocking under packet loss.
- Misconfigured timeouts causing retries and load spikes.
- Long-lived connections without keepalives leading to silent drops.

## Observability (How to detect issues)
- **Metrics:** retransmission rate, connection errors, handshake latency.
- **Logs:** connection resets, timeout errors.
- **Traces:** spikes in network time between spans.
- **Alerts:** elevated retransmits or connection failures.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Set reasonable connect and read timeouts
  - [ ] Use keepalives for long-lived connections
- **Security / Compliance notes**
  - Use [TLS](tls.md) for encryption at the application layer.
- **Performance notes**
  - Watch for Nagle vs latency-sensitive workloads.
- **Operational notes**
  - Monitor SYN backlog and connection limits.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Relying on infinite retries over TCP.
  - **Why it’s bad:** amplifies load during network incidents.
  - **Better approach:** bounded retries with backoff and timeouts.

## Interview readiness
### “Explain it like I’m teaching”
- TCP sets up a connection and guarantees ordered delivery by acknowledging packets and retransmitting losses. It’s reliable but adds latency and overhead.

### Trap questions (with answers)
1) **Q:** Does TCP guarantee zero loss?
   - **A:** No; it detects loss and retransmits, but loss can still occur and add delay.
2) **Q:** Is TCP message-oriented?
   - **A:** No; it’s a byte stream without message boundaries.
3) **Q:** Is TCP always faster than UDP?
   - **A:** No; TCP’s reliability and ordering add overhead.

### Quick self-check (5 items)
- [ ] I can define TCP’s guarantees in 2–3 sentences.
- [ ] I can state when to use it and when not to.
- [ ] I can explain at least 2 trade-offs.
- [ ] I can give a concrete example from memory.
- [ ] I can name 1 failure mode and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Networking basics](networking-basics.md)
- [Network layers (OSI & TCP/IP)](network-layers.md)

### Related topics
- [UDP](udp.md)
- [HTTP](http.md)
- [TLS](tls.md)
- [API design basics](../system-design/api-design-basics.md)

### Compare with
- [UDP](udp.md) — reliable ordered stream vs best-effort datagrams.

## Notes / Inbox (optional)
- N/A
