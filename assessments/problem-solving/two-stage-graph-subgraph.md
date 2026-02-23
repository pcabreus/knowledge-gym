---
title: Two-Stage Retrieval — Large Graph Subgraph Retrieval
created_at: 2026-02-23
---

1) Summary: Retrieve a relevant subgraph by first selecting seed nodes with a cheap filter (partition, LSH on features, or label index), then expand using exact graph traversal to collect neighbors and compute final subgraph.

2) Clarifying questions
- Is the graph directed or undirected?  
- Are node features available for approximate indexing (embeddings, labels)?  
- Expected subgraph size limit and latency requirements?

3) Possible approaches
- Full graph traversal per query: exact but potentially very expensive.
- Two-stage: seed selection (partition/LSH/label-index) -> bounded BFS/DFS expansion. Pros: bounded cost. Cons: may miss long-range dependencies if seeds insufficient.
- Graph summary/index: maintain precomputed neighborhoods for hot nodes (materialized). Faster, more operational overhead.

4) Pseudocode
```
seeds = seed_index.lookup(query)
subgraph = set()
for s in seeds:
  subgraph |= bounded_bfs(s, depth_limit)
final = exact_filter(subgraph, query)
return final
```

5) Implementation: Python (toy partition + bounded BFS)
```python
from collections import defaultdict, deque

Graph = defaultdict(list)  # adjacency list
PartitionIndex = defaultdict(set)  # coarse mapping from partition key to node ids

def bounded_bfs(start, depth_limit=2):
    q=deque([(start,0)])
    seen={start}
    result=[]
    while q:
        node, d = q.popleft()
        result.append(node)
        if d<depth_limit:
            for nbr in Graph[node]:
                if nbr not in seen:
                    seen.add(nbr)
                    q.append((nbr,d+1))
    return set(result)

def query_subgraph(partition_keys, depth_limit=2):
    seeds=set()
    for pk in partition_keys:
        seeds |= PartitionIndex.get(pk, set())
    sub=set()
    for s in seeds:
        sub |= bounded_bfs(s, depth_limit)
    # exact filtering step (e.g., nodes with attribute matching)
    final = [n for n in sub if exact_node_filter(n)]
    return final

def exact_node_filter(node_id):
    # placeholder: implement application-specific checks
    return True
```

6) Complexity
- Seed lookup: depends on index (O(1) per partition key expected).  
- Expansion: O(|V_sub|+|E_sub|) where V_sub/E_sub are nodes/edges in BFS neighborhood; controlled by depth_limit.

7) Test cases
- Small graph exact match: seed covers full subgraph.
- Seedless query: index returns no seeds (should return empty result).  
- Deep dependency: verify whether depth_limit misses expected nodes.
- Directed graph edge cases: unreachable nodes.
- Large-degree node: ensure expansion budget avoids explosion.

8) Optimizations and variants
- Use LSH over node embeddings for semantic seed selection.
- Precompute and cache k-hop neighborhoods for hot nodes.
- Adaptive depth: increase depth when seed coverage low.

9) Interview questions
- How would you bound expansion to avoid explosion for high-degree nodes?
- How to keep seed index consistent with graph updates?
- How to measure recall and precision for the two-stage retrieval in graphs?
