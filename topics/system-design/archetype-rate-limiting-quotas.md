---
id: archetype-rate-limiting-quotas
title: "Rate Limiting & Quotas"
type: pattern
status: learning
importance: 90
difficulty: medium
tags: [system-design, rate-limiting, throttling, quotas, token-bucket, leaky-bucket]
canonical: true
aliases: [throttling, admission-control, usage-limits, api-quota]
created_at: 2026-01-28
updated_at: 2026-01-28
---

# Rate Limiting & Quotas

## TL;DR (BLUF)
- Limit requests per identity (user/IP/token) over time to protect systems from abuse and overload.
- Key challenge: distributed correctness (global limits across nodes), high QPS overhead, fairness vs precision.
- Solution patterns: token bucket / leaky bucket, centralized counter store (Redis), adaptive throttling.

## Definition
**What it is:** A system archetype that enforces limits on request rates or resource usage per identity (user, API key, IP) to prevent abuse, ensure fairness, and protect system stability.

**Key terms:**
- **Rate limit**: Maximum requests allowed per time window (e.g., 100 req/min)
- **Quota**:累加的 usage limit (e.g., 10GB storage, 1M API calls/month)
- **Token bucket**: Algorithm allowing bursts with refill rate
- **Leaky bucket**: Algorithm enforcing smooth rate
- **Sliding window**: Time window that slides with each request
- **Fixed window**: Time resets at fixed intervals (00:00, 01:00)

## Why it matters
- **Protection**: Prevents abusive clients from causing outages for all users
- **Fairness**: Ensures resources are distributed equitably
- **Cost control**: Prevents runaway costs from unbounded usage
- **Interview frequency**: Common in API gateway, SaaS, and infrastructure design

**Typical real-world consequences of getting it wrong:**
- Public API: One abusive client saturates CPU → outages for all customers
- No rate limiting on login: Brute-force attacks succeed, credential stuffing
- Incorrect distributed counting: User bypasses limits by hitting multiple nodes

## Scope & Non-goals
**In scope:**
- Algorithms (token bucket, leaky bucket, sliding window)
- Distributed enforcement (multi-node correctness)
- Per-key vs global limits
- Adaptive throttling based on system health

**Out of scope / NOT solved by this:**
- Authentication/authorization → see [Security & Access Control](archetype-security-access-control.md)
- DDoS protection at network layer → separate concern (firewall, WAF)
- Backpressure within services → see [Backpressure](backpressure.md)

## Mental model / Intuition
Think of it like **water flowing through a faucet**:
- **Token bucket**: Bucket holds tokens (capacity). Tokens refill at steady rate. Each request takes a token. Allows bursts (if bucket full) but constrains long-term rate.
- **Leaky bucket**: Requests enter bucket. Bucket leaks at fixed rate. If bucket overflows, requests are rejected. Smooths out bursts.
- **Fixed window**: Like an hourglass that resets every hour. Once sand runs out, no more requests until next hour.
- **Sliding window**: Rolling count over last N seconds; more accurate but complex.

## Decision rules (When to use / When not to use)
### Use it when
- **Public APIs**: Exposed to internet; abuse risk high
- **Expensive operations**: LLM calls, video encoding, report generation
- **Multi-tenancy**: Prevent one tenant from starving others
- **Scarce resources**: Protect limited downstream capacity

### Avoid it when
- **Trusted internal services**: Over-complicates; other safeguards exist
- **Ultra-low latency required**: Rate limit check adds latency (typically < 1ms but still)
- **No abuse risk**: Private, controlled environment

## How I would use it (practical)
### Context: Public API for image processing
**Problem:** API exposed to internet. Without limits, one abusive client can saturate CPU → outage for all. Need to enforce 1000 req/min per API key.

**Steps:**
1. **Choose algorithm**: Token bucket (allows small bursts, smooth long-term rate)
2. **Distributed store**: Redis (centralized counter for correctness across nodes)
3. **Implementation**:
   - Key: `rate_limit:{api_key}`
   - Atomic ops: `INCR` + `EXPIRE` or Lua script for token bucket logic
   - On each request: check tokens available; if yes, decrement and proceed; if no, return 429 Too Many Requests
4. **Granularity**: Per API key + per IP (fallback for unauthenticated)
5. **Response headers**: `X-RateLimit-Limit: 1000`, `X-RateLimit-Remaining: 847`, `X-RateLimit-Reset: 1643723400`
6. **Adaptive throttling**: If downstream CPU > 80%, reduce limits by 50%

**What success looks like:**
- No single client can cause outage (hard limit enforced)
- Legitimate bursts allowed (token bucket flexibility)
- p99 rate limit check latency < 5ms

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Protects system from abuse and overload
  - Fair resource distribution
  - Simple to explain to users (clear limits)
- **Cons / Risks:**
  - Adds latency to every request (typically small)
  - Distributed correctness is hard (race conditions)
  - Edge cases: bursty legitimate traffic may hit limits
  - Operational overhead (rate limit store, monitoring)

### Alternatives
- **No rate limiting (trust users):** Simple but risky
  - **When better:** Fully trusted internal systems, low abuse risk
- **Coarse global throttling:** Simpler but not per-identity fairness
  - **When better:** Only need system-level protection, not fairness
- **Usage billing (pay-per-use):** Economic disincentive instead of hard limit
  - **When better:** SaaS model where overages are acceptable with payment

**How to choose:**
- Public APIs → always rate limit (start with token bucket)
- Internal services → consider if abuse risk exists; else skip
- Multi-tenant SaaS → rate limit + quotas (per tenant fairness)

## Failure modes & Pitfalls
### Where it hurts (why it hurts)
1. **Distributed correctness (global limits across many nodes)**
   - **Symptom:** User bypasses limit by hitting different servers; limit is per-node not global
   - **Why:** Each node maintains separate counter; no coordination
   - **Solution:** Centralized counter store (Redis) with atomic ops

2. **High QPS and memory usage**
   - **Symptom:** Rate limit store overwhelmed; high latency or OOM
   - **Why:** Tracking millions of users/IPs independently
   - **Solution:** TTL-based expiry, approximate algorithms (probabilistic), tiered limits (global + per-key)

3. **Fairness vs precision**
   - **Symptom:** Fixed window allows 2× burst at window boundary (999 at 00:59, 1000 at 01:00)
   - **Why:** Fixed window resets; no memory of previous window
   - **Solution:** Sliding window (more accurate) or sliding log (precise but expensive)

4. **Legitimate bursts blocked**
   - **Symptom:** User hits limit during normal usage spike (e.g., batch job)
   - **Why:** Limit too strict or wrong algorithm (leaky bucket too smooth)
   - **Solution:** Token bucket (allows bursts) + communicate limits to users

## Observability (How to detect issues)
### Metrics
- `rate_limit_rejections_per_sec`: Requests rejected (429 responses)
- `rate_limit_check_latency_p99`: Overhead of rate limit check
- `top_rate_limited_keys`: Which users/IPs hit limits most
- `rate_limit_store_errors`: Redis failures

### Logs
- Log 429 responses: key, limit, current usage, timestamp

### Traces
- Span: `rate_limit_check` (should be < 5ms)

### Alerts
- `rate_limit_rejections > 10% of traffic` → limits too aggressive or attack
- `rate_limit_check_latency_p99 > 50ms` → store overloaded
- `rate_limit_store_errors > 1%` → Redis availability issue

## Implementation notes
### Checklist
- [ ] Choose algorithm (token bucket for most use cases)
- [ ] Select distributed store (Redis recommended)
- [ ] Define limits per identity type (user, IP, API key)
- [ ] Implement atomic counter ops (Lua script or Redis commands)
- [ ] Return 429 with Retry-After header
- [ ] Add response headers (X-RateLimit-*)
- [ ] Monitor rejections and latency
- [ ] Plan adaptive throttling based on system load

### Performance notes
- **Algorithm choice**: Token bucket = best balance; sliding window = more accurate but expensive
- **Store latency**: Redis typical p99 < 1ms; use pipelining if checking multiple limits

### Operational notes
- **Key expiry**: Use TTL to auto-clean old keys (avoid memory leak)
- **Failover**: If rate limit store is down, choose: fail open (allow all) or fail closed (reject all). Most choose fail open with monitoring.

## Mini example (code)
```python
# Token bucket with Redis (simplified)
import redis
import time

redis_client = redis.Redis()

def is_rate_limited(api_key, limit=100, window=60):
    """Token bucket: allow 100 requests per 60 seconds."""
    key = f"rate_limit:{api_key}"
    
    # Lua script for atomic token bucket check
    lua_script = """
    local key = KEYS[1]
    local limit = tonumber(ARGV[1])
    local window = tonumber(ARGV[2])
    local now = tonumber(ARGV[3])
    
    local current = redis.call('GET', key)
    if current == false then
        redis.call('SET', key, limit - 1, 'EX', window)
        return 0  -- Not limited
    end
    
    current = tonumber(current)
    if current > 0 then
        redis.call('DECR', key)
        return 0  -- Not limited
    else
        return 1  -- Limited
    end
    """
    
    result = redis_client.eval(lua_script, 1, key, limit, window, int(time.time()))
    return result == 1

# Usage in API handler
def handle_request(api_key):
    if is_rate_limited(api_key):
        return {"error": "Rate limit exceeded"}, 429, {
            "Retry-After": "60",
            "X-RateLimit-Limit": "100",
            "X-RateLimit-Remaining": "0"
        }
    
    # Process request
    return process_request()
```

## Common anti-patterns
### Anti-pattern: Fixed window without sliding
- **What people do:** Count requests in fixed 1-minute windows (00:00-01:00, 01:00-02:00)
- **Why it's bad:** Allows 2× burst at boundary (999 at 00:59, 1000 at 01:00 = 1999 in 1 minute)
- **Better approach:** Sliding window or token bucket

### Anti-pattern: Per-node limits instead of global
- **What people do:** Each server enforces limit independently
- **Why it's bad:** User can bypass by hitting different servers (10 servers × 100 limit/server = 1000 total)
- **Better approach:** Centralized counter (Redis) for global enforcement

### Anti-pattern: No rate limit headers
- **What people do:** Return 429 without context
- **Why it's bad:** Clients don't know when to retry or how much quota remains
- **Better approach:** Include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After` headers

### Anti-pattern: Rate limiting expensive operations same as cheap ones
- **What people do:** Same limit for GET /user and POST /generate_report
- **Why it's bad:** Expensive ops should have stricter limits
- **Better approach:** Tiered limits per endpoint or operation cost (cost-based rate limiting)

## Interview readiness
### "Explain it like I'm teaching"
Rate limiting protects your system by enforcing a maximum number of requests per user or API key over time. We typically use a token bucket algorithm: imagine a bucket that holds tokens, refilling at a steady rate (e.g., 100 tokens per minute). Each request takes a token. If the bucket is empty, we reject the request with a 429 status. This allows small bursts (if the bucket is full) while constraining long-term rate. For distributed systems, we use a centralized counter (like Redis) to enforce limits globally across all servers.

### Trap questions (with answers)
1) **Q:** What's the difference between token bucket and leaky bucket?
   - **A:** Token bucket: Tokens refill at a rate; requests consume tokens; allows bursts if bucket full. Leaky bucket: Requests enter a queue that "leaks" (processes) at a fixed rate; smooths bursts. Token bucket is more flexible (allows bursts), leaky bucket enforces strict smoothness.

2) **Q:** How do you enforce rate limits in a distributed system with multiple servers?
   - **A:** Use a centralized counter store like Redis with atomic operations. Each server checks and increments the counter in Redis before allowing a request. This ensures a global limit across all servers, not a per-server limit.

3) **Q:** What's the problem with fixed-window rate limiting?
   - **A:** Allows 2× burst at window boundaries. If limit is 1000/min, a user can send 1000 at 00:59:59 and another 1000 at 01:00:00, totaling 2000 in 1 second. Sliding window or token bucket avoids this.

4) **Q:** What happens if the rate limit store (Redis) goes down?
   - **A:** Two options: (1) Fail open—allow all requests (no rate limiting during outage). (2) Fail closed—reject all requests. Most systems fail open to preserve availability, with monitoring/alerting for when rate limiting is disabled.

5) **Q:** How do you handle legitimate bursts (e.g., a batch job that needs 500 requests quickly)?
   - **A:** Use token bucket algorithm (allows bursts). Alternatively, provide "burst quota" separate from sustained rate, or allow users to request temporary limit increases. Document limits clearly so users can design around them.

## Links
**Part of:**
- [System Design Archetypes](system-design-archetypes.md)

**Prerequisites:**
- [Rate Limiting](rate-limiting.md)
- [Redis](../databases/redis.md)

**Related archetypes:**
- [Distributed Caching](archetype-distributed-caching.md) — Redis as shared state
- [Multi-Tenancy & Isolation](archetype-multi-tenancy.md) — per-tenant quotas
- [Security & Access Control](archetype-security-access-control.md) — authentication before rate limiting

**Related topics:**
- [API Gateway](api-gateway.md)
- [Backpressure](backpressure.md) — internal throttling
