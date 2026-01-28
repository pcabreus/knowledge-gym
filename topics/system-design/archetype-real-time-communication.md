---
id: archetype-real-time-communication
title: "Real-Time Communication (WebSockets / Presence)"
type: pattern
status: learning
importance: 80
difficulty: hard
tags: [system-design, websockets, real-time, presence, push, pub-sub]
canonical: true
aliases: [real-time-messaging, presence, pub-sub-over-sockets]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Real-Time Communication (WebSockets / Presence)

## TL;DR
- Low-latency updates to clients via stateful connections (WebSockets, Server-Sent Events).
- Key challenges: connection scalability (millions of sockets), presence correctness, message delivery when offline.
- Solutions: gateway layer managing sockets, pub-sub backbone distributing messages, store-and-forward + retries.

## Where it hurts (why it hurts)
1. **Connection scalability (millions of sockets)**: Each connection holds memory/state → OOM
   - **Solution**: Gateway layer (dedicated connection servers), horizontal scaling, connection limits per user
2. **Presence correctness**: Network drops cause false "offline"; heartbeats missed
   - **Solution**: Heartbeat protocol with timeout; exponential backoff; "away" vs "offline" states
3. **Message delivery when offline**: User disconnects → messages lost
   - **Solution**: Store-and-forward (persist messages while offline), replay on reconnect

## Decision rules
- **Use when**: Chat, collaborative editing, live dashboards, gaming, notifications
- **Avoid when**: Polling sufficient (updates every 30s+), no need for push

## Trade-offs
- **WebSockets (persistent)**: True bidirectional, low latency; connection overhead
- **Server-Sent Events (SSE)**: Simpler (HTTP), one-way; easier to implement
- **Polling (fallback)**: No persistent connection; higher latency, more requests
- **Choose**: Bidirectional + low latency → WebSockets; one-way push → SSE; legacy browsers → polling

## Explicit example
Chat app: Users expect instant delivery + "online/offline" indicators. WebSocket connection per user. Gateway servers manage connections (sticky sessions). Message published to pub/sub (Redis/Kafka); gateway subscribes and pushes to connected clients. Heartbeat every 30s; timeout after 60s → mark offline. Store undelivered messages; replay on reconnect.

## Links
**Part of**: [System Design Archetypes](system-design-archetypes.md)  
**Related**: [Fan-Out](archetype-fan-out.md), [High-Throughput Event Ingestion](archetype-event-ingestion.md)
