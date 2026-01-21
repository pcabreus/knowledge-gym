---
id: tls
title: "TLS (Transport Layer Security)"
type: technology
status: learning
importance: 55
difficulty: medium
tags: [operations, networking, security]
canonical: true
aliases: [ssl]
created_at: 2026-01-21
updated_at: 2026-01-21
---

# TLS (Transport Layer Security)

## TL;DR (BLUF)
- TLS encrypts data in transit and authenticates endpoints.
- Use it for any network traffic carrying sensitive or user data.
- Trade-off: extra handshake latency and operational complexity with certificates.

## Definition
**What it is:** A security protocol that provides confidentiality, integrity, and (optionally) mutual authentication for network connections, typically on top of [TCP](tcp.md).  
**Key terms:** certificate, handshake, cipher suite, PKI, mTLS.

## Why it matters
- It prevents eavesdropping and tampering on network traffic.
- Certificate and handshake failures are common sources of outages.

## Scope & Non-goals
**In scope:** TLS purpose, handshake flow, and operational trade-offs.  
**Out of scope / NOT solved by this:** application-level auth and authorization.

## Mental model / Intuition
- TLS is a secure envelope: it wraps your traffic so only intended parties can read it.

## Decision rules (When to use / When not to use)
### Use it when
- Traffic crosses untrusted networks (internet, shared networks).
- You need to verify the server identity.
### Avoid it when
- Only in rare, fully isolated environments with no sensitive data (and you accept the risk).

## How I would use it (practical)
- **Context:** Public [HTTP](http.md) APIs and internal service-to-service traffic.
- **Steps:**
  1) Issue certificates (CA or managed). 
  2) Configure TLS termination at load balancer or service.
  3) Rotate certificates and monitor handshake errors.
- **What success looks like:** encrypted traffic, low handshake failures, smooth rotation.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** encryption in transit, integrity, endpoint authentication.
- **Cons / Risks:** handshake latency, cert expiration incidents, and config complexity.
### Alternatives
- **Private networking only:** reduces exposure but doesn’t replace encryption.
- **How to choose:** default to TLS; exceptions should be rare and documented.

## Failure modes & Pitfalls
- Expired certificates causing outages.
- Mismatched TLS versions or ciphers.
- Termination at the edge without end-to-end encryption where required.

## Observability (How to detect issues)
- **Metrics:** handshake success rate, TLS version distribution, cert expiry days.
- **Logs:** handshake failures, cert validation errors.
- **Traces:** connection setup time spikes.
- **Alerts:** upcoming cert expiry or handshake error spikes.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Automate cert issuance and rotation
  - [ ] Enforce TLS version policies
- **Security / Compliance notes**
  - Use mTLS for sensitive internal traffic where identity matters.
- **Performance notes**
  - Enable session resumption to reduce handshake latency.
- **Operational notes**
  - Track cert expiry across environments.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Manual certificate renewal.
  - **Why it’s bad:** expired certs cause outages.
  - **Better approach:** automate renewal and alert ahead of expiry.

## Interview readiness
### “Explain it like I’m teaching”
- TLS encrypts traffic and verifies identities using certificates. It protects data in transit but adds handshake latency and certificate management overhead.

### Trap questions (with answers)
1) **Q:** Is TLS the same as HTTPS?
  - **A:** No; HTTPS is [HTTP](http.md) over TLS.
2) **Q:** Does TLS guarantee authorization?
   - **A:** No; it only secures the connection and may authenticate endpoints.
3) **Q:** Is mTLS always required?
   - **A:** No; use it when you need mutual authentication.

### Quick self-check (5 items)
- [ ] I can explain what TLS provides (confidentiality, integrity, auth).
- [ ] I can state when to use it and when not to.
- [ ] I can describe certificate lifecycle risks.
- [ ] I can name 1 failure mode and how to detect it.
- [ ] I can explain HTTPS as HTTP over TLS.

## Links (NO duplication)
### Prerequisites
- [Network layers (OSI & TCP/IP)](network-layers.md)
- [TCP](tcp.md)

### Related topics
- [HTTP](http.md)
- [API design basics](../system-design/api-design-basics.md)

### Compare with
- [HTTP](http.md) — protocol semantics vs transport security.

## Notes / Inbox (optional)
- N/A
