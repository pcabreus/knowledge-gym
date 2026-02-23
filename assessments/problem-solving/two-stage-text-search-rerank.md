---
title: Two-Stage Retrieval — Approximate Text Search + Exact Re-rank
created_at: 2026-02-23
---

1) Summary: Use a fast approximate text candidate generator (e.g., n-gram index or ANN) to produce candidates, then apply exact scoring and reranking to produce final results.

2) Clarifying questions
- Is text matching fuzzy (typos, synonyms) or exact?  
- Expected corpus size and latency target?  
- Are semantic vectors available or only raw text?

3) Possible approaches
- Brute force exact scoring: precise but O(n) per query.
- Two-stage with n-gram inverted index: low-cost candidate set then exact scoring. Good when queries are many.
- Vector ANN + exact rerank: semantic search; uses embeddings (requires embedding pipeline).

4) Pseudocode
```
candidates = approx_index.query(q, k=100)
scored = [(c, exact_score(c, q)) for c in candidates]
return top_k(sorted(scored, key=score), k)
```

5) Implementation: Python (toy trigram inverted index)
```python
from collections import defaultdict, Counter
from typing import List, Tuple

def trigrams(s: str):
    s = f'  {s} '
    return [s[i:i+3] for i in range(len(s)-2)]

class TrigramIndex:
    def __init__(self):
        self.inv = defaultdict(set)
        self.docs = {}

    def add(self, doc_id: str, text: str):
        self.docs[doc_id] = text
        for t in set(trigrams(text)):
            self.inv[t].add(doc_id)

    def query(self, q: str, k: int=50) -> List[str]:
        # Stage 1: candidate generation via trigram overlap
        cnt = Counter()
        for t in set(trigrams(q)):
            for doc in self.inv.get(t, ()): cnt[doc]+=1
        # return top candidates by overlap
        return [doc for doc,_ in cnt.most_common(k)]

def exact_score(doc_text: str, q: str) -> int:
    # example: longest common substring length as a simple exact score
    if q in doc_text: return len(q)*10
    return sum(1 for _ in set(trigrams(q)) & set(trigrams(doc_text)))

def search(idx: TrigramIndex, q: str, k: int=10):
    candidates = idx.query(q, k=200)
    scored = [(doc, exact_score(idx.docs[doc], q)) for doc in candidates]
    scored.sort(key=lambda x: x[1], reverse=True)
    return scored[:k]

# Small example left to runner
```

6) Complexity
- Index building: O(total_chars) to build trigrams.  
- Query Stage 1: O(#trigrams_in_q + total_postings_for_these_trigrams) but typically small.  
- Stage 2: O(c * exact_cost) where c = candidates. Overall cost controlled by candidate budget c.

7) Test cases
- Exact match should rank top.  
- Query with typo: trigram overlap should still return candidates (depending on typo severity).
- Large corpus: verify candidate count stays bounded and latency acceptable.
- Empty query returns empty list.  
- Query with no overlap returns empty candidates.

8) Optimizations and variants
- Use locality-sensitive hashing or vector embeddings in stage 1 for semantic search.
- Dynamic candidate budget: increase when first-stage recall is low.
- Cache popular query results for instant responses.

9) Interview follow-ups
- How would you support phrase and proximity scoring in the exact stage?
- What diagnostics would you add to detect recall regressions?
- How to extend to multilingual corpora or tokenization differences?
