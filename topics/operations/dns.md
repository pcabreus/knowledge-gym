---
id: dns
title: "DNS (Domain Name System)"
type: technology
status: learning
importance: 50
difficulty: medium
tags: [operations, networking]
canonical: true
aliases: []
created_at: 2026-01-21
updated_at: 2026-01-21
---

# DNS (Domain Name System)

## TL;DR (BLUF)
- DNS maps human-readable names to IP addresses and other records.
- Use it to route traffic and decouple services from IP changes.
- Trade-off: caching and propagation delays can cause stale routing.

## Definition
**What it is:** A distributed naming system that resolves domain names to resource records (A/AAAA, CNAME, TXT, MX, etc.).  
**Key terms:** resolver, authoritative server, TTL, cache, A/AAAA, CNAME.

## Why it matters
- DNS failures look like outages even when services are healthy.
- TTLs and caching affect rollout speed and failover behavior.

## Scope & Non-goals
**In scope:** name resolution flow, TTLs, caching, and operational trade-offs.  
**Out of scope / NOT solved by this:** load balancing algorithms or CDN design.

## Mental model / Intuition
- DNS is a phonebook: you ask for a name and get a number (IP).
- Caches speed lookups but can keep old numbers for a while.

## Decision rules (When to use / When not to use)
### Use it when
- You need stable names for changing infrastructure.
- You need to route traffic across regions or providers.
### Avoid it when
- You require instant cutovers (DNS caching can delay).

## How I would use it (practical)
- **Context:** Blue/green deployment cutover.
- **Steps:**
  1) Set a low TTL before planned changes.
  2) Update DNS records during cutover.
  3) Monitor resolution and error rates.
- **What success looks like:** predictable propagation and minimal user impact.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:** decouples service identity from IPs; global reach.
- **Cons / Risks:** caching delays, resolver variability, and partial propagation.
### Alternatives
- **Load balancer cutover:** faster and more controllable for traffic switching.
- **How to choose:** use DNS for stable naming and coarse routing, not instant failover.

## Failure modes & Pitfalls
- Misconfigured records causing traffic black holes.
- Stale caches preventing rapid rollback.
- Split-horizon DNS leading to inconsistent routing.

## Observability (How to detect issues)
- **Metrics:** resolution latency, NXDOMAIN rate, cache hit ratio.
- **Logs:** resolver errors, record changes.
- **Traces:** increased latency before app-level timeouts.
- **Alerts:** spikes in NXDOMAIN or resolution failures.

## Implementation notes (if applicable)
- **Checklist**
  - [ ] Define TTLs per record type
  - [ ] Automate DNS changes with audit trails
- **Security / Compliance notes**
  - Protect DNS changes with strong access control.
- **Performance notes**
  - Use local resolvers or caching to reduce latency.
- **Operational notes**
  - Document rollback steps and expected propagation time.

## Mini example (if applicable)
N/A

## Common anti-patterns
- **Anti-pattern:** Using DNS for instant cutover.
  - **Why it’s bad:** caching delays make the switch slow and inconsistent.
  - **Better approach:** use a load balancer or service mesh for fast switches.

## Interview readiness
### “Explain it like I’m teaching”
- DNS maps names to IPs using a distributed, cached system. It’s essential for routing but can introduce delays due to caching.

### Trap questions (with answers)
1) **Q:** Does lowering TTL guarantee instant propagation?
   - **A:** No; caches may still hold old records and resolvers behave differently.
2) **Q:** Is DNS highly consistent globally?
   - **A:** No; it’s eventually consistent due to caching and propagation.
3) **Q:** Can DNS replace a load balancer?
   - **A:** Not for instant failover; DNS is slower and less controllable.

### Quick self-check (5 items)
- [ ] I can explain how DNS resolution works end-to-end.
- [ ] I can describe TTL trade-offs.
- [ ] I can name 1 failure mode and how to detect it.
- [ ] I can explain why DNS cutovers are slow.
- [ ] I can relate DNS to availability.

## Links (NO duplication)
### Prerequisites
- [Networking basics](networking-basics.md)
- [Network layers (OSI & TCP/IP)](network-layers.md)

### Related topics
- [TCP](tcp.md)
- [HTTP](http.md)
- [Blue/green deployments](blue-green-deployments.md)

### Compare with
- (TODO) Load balancing — DNS routing vs L4/L7 traffic control.

## Notes / Inbox (optional)
- N/A
