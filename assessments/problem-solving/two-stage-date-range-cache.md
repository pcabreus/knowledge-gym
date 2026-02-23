---
title: Two-Stage Retrieval — Date-Range Cache (Problem)
created_at: 2026-02-23
---

1) Summary: Use a two-stage retrieval for date-range queries: coarse daily buckets in cache, then in-memory filter for minute/second precision.

2) Clarifying questions to ask the interviewer
- What are typical query time ranges (minutes, hours, days)?
- How fresh must results be (strictly consistent vs eventual)?
- Approximate dataset size and memory constraints for in-memory filtering?

3) Possible approaches
- Full exact range index: precise but expensive to maintain and query at scale.
- Two-stage (recommended): coarse daily buckets in cache → in-memory filter for precise timestamps. Pros: fewer reads, low latency. Cons: added complexity, careful bucket sizing needed.
- Materialized per-query views: fast but high maintenance cost.

4) Pseudocode
```
// Stage 1: coarse retrieval from cache by day buckets
start_day, end_day = normalize_to_days(start_ts, end_ts)
candidates = set()
for day in range(start_day, end_day):
  candidates += cache.get(day_bucket_key(day))

// Stage 2: exact in-memory filter
results = [r for r in candidates if start_ts <= r.ts <= end_ts]
return sort_by_timestamp(results)
```

5) Implementation: Python (3.10+)
```python
from datetime import datetime, timedelta
from collections import defaultdict

# Simple in-memory cache keyed by day string -> list of records
Cache = defaultdict(list)

def day_key(ts: datetime) -> str:
    return ts.strftime("%Y-%m-%d")

def add_record(rec):
    # rec must have rec['ts'] as datetime
    Cache[day_key(rec['ts'])].append(rec)

def query_range(start_ts: datetime, end_ts: datetime):
    # Stage 1: gather day buckets
    day = start_ts.replace(hour=0,minute=0,second=0,microsecond=0)
    candidates = []
    while day <= end_ts:
        candidates.extend(Cache.get(day_key(day), []))
        day += timedelta(days=1)

    # Stage 2: filter precisely in-memory
    results = [r for r in candidates if start_ts <= r['ts'] <= end_ts]
    results.sort(key=lambda r: r['ts'])
    return results

# Example usage and edge handling left as exercises
```

6) Complexity
- Stage 1: O(d) where d = number of day-buckets intersected (usually small).
- Stage 2: O(c) where c = candidates returned by coarse stage. Total time O(d + c).
- Space: depends on bucketization; storing daily buckets is O(n) total data.

7) Test cases
- Query within a single day with multiple records (should return only those exact timestamps).
- Query spanning many days but sparse data (candidate set small).
- Query spanning many days with dense data (verify candidate threshold/alerting).
- Empty dataset returns empty list.
- Edge: records exactly at start_ts or end_ts should be included.

8) Possible optimizations and variants
- Use hourly buckets if precision requires smaller candidate sets.
- Maintain per-bucket summary (min_ts, max_ts, count) to skip empty/irrelevant buckets.
- Add bloom filters per bucket to quickly test non-membership.
- Hybrid: keep most-recent N days in memory, older buckets in a distributed cache.

9) Interview questions and follow-ups
- How would you size buckets (day vs hour) for different workloads?
- How do you handle cache invalidation and writes across multiple writers?
- How would you monitor and alert if candidate counts exceed expected thresholds?
