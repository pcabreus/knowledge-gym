# Detect Cycles in a Directed Graph

## Definition
Given a directed graph, determine if it contains any cycles (circular dependencies).

## Why it matters
Fundamental for dependency analysis, build systems, and many interview questions.

## When to use / not to use
Use for dependency graphs, import cycles, scheduling. Not for undirected graphs (use a different approach).

## Trade-offs
- DFS is simple and efficient for sparse graphs
- Kahn's algorithm (topological sort) is robust for large graphs

## Step-by-step solution
### 1. Clarifying questions
- How is the graph represented? (adjacency list/matrix)
- Can there be self-loops?
- Are there disconnected nodes?

### 2. Approaches
- DFS with color marking (white/gray/black)
- Kahn's algorithm (topological sort, count in-degrees)

### 3. Pseudocode (DFS)
```
visited = [0] * n  # 0=unvisited, 1=visiting, 2=visited
for each node:
    if dfs(node): return True
return False

def dfs(node):
    if visited[node] == 1: return True  # found cycle
    if visited[node] == 2: return False
    visited[node] = 1
    for neighbor in graph[node]:
        if dfs(neighbor): return True
    visited[node] = 2
    return False
```

### 4. Implementations
#### Python
```python
def has_cycle(graph: dict[int, list[int]]) -> bool:
    n = len(graph)
    visited = [0] * n
    def dfs(node):
        if visited[node] == 1:
            return True
        if visited[node] == 2:
            return False
        visited[node] = 1
        for neighbor in graph.get(node, []):
            if dfs(neighbor):
                return True
        visited[node] = 2
        return False
    for node in graph:
        if dfs(node):
            return True
    return False
```
#### Go
```go
func hasCycle(graph map[int][]int) bool {
    n := len(graph)
    visited := make([]int, n)
    var dfs func(int) bool
    dfs = func(node int) bool {
        if visited[node] == 1 {
            return true
        }
        if visited[node] == 2 {
            return false
        }
        visited[node] = 1
        for _, neighbor := range graph[node] {
            if dfs(neighbor) {
                return true
            }
        }
        visited[node] = 2
        return false
    }
    for node := range graph {
        if dfs(node) {
            return true
        }
    }
    return false
}
```
#### TypeScript
```typescript
function hasCycle(graph: Record<number, number[]>): boolean {
    const n = Object.keys(graph).length;
    const visited: number[] = Array(n).fill(0);
    function dfs(node: number): boolean {
        if (visited[node] === 1) return true;
        if (visited[node] === 2) return false;
        visited[node] = 1;
        for (const neighbor of graph[node] || []) {
            if (dfs(neighbor)) return true;
        }
        visited[node] = 2;
        return false;
    }
    for (const node in graph) {
        if (dfs(Number(node))) return true;
    }
    return false;
}
```

### 5. Complexity
- Time: O(V+E)
- Space: O(V)

### 6. Test cases
- Acyclic: {0: [1], 1: [2], 2: []} → False
- Simple cycle: {0: [1], 1: [2], 2: [0]} → True
- Self-loop: {0: [0]} → True
- Disconnected: {0: [1], 1: [], 2: [3], 3: []} → False
- Empty graph: {} → False

### 7. Optimizations/Variants
- Use Kahn's algorithm for large graphs
- Detect all cycles, not just existence
- Handle graphs with string node labels

### 8. Interview follow-ups
- How to return the actual cycle?
- How to handle very large graphs?
- How to adapt for undirected graphs?

## Prerequisites
- [Graph theory](../_index.md)
- [DFS](../_index.md)

## Related topics
- [Topological sort](../_index.md)
- [Dependency analysis](../_index.md)

## Trap questions
1. What if the graph is empty?
   - No cycles.
2. What if there are self-loops?
   - They count as cycles.
3. How to handle non-integer node labels?
   - Use a mapping from labels to indices.
