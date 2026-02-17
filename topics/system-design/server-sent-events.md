---
title: Server-Sent Events (SSE)
---

# Server-Sent Events (SSE)

## Definition
Server-Sent Events (SSE) is a technology that allows a server to push real-time updates to clients over a single HTTP connection in a unidirectional way.

## Why it matters
It enables near real-time updates for dashboards, notifications, and other use cases without polling or WebSockets.

## When to use / when not to use
Use for unidirectional, real-time updates from server to client. Not suitable for bidirectional communication (use WebSockets).

## Trade-offs
- + Simple, low-overhead for real-time updates
- + Works over standard HTTP
- - Not bidirectional
- - Can be affected by proxies/timeouts

## Prerequisites
- [HTTP](../operations/http.md)

## Related topics
- [WebSockets](websockets.md) (TODO if missing)
- [API Gateway](api-gateway.md)

## Trap questions
1. What is SSE?
   - A way for servers to push real-time updates to clients over HTTP.
2. When should you avoid SSE?
   - When you need bidirectional communication.
3. What is a common pitfall with SSE?
   - Proxy or timeout issues breaking the connection.
