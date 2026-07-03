---
name: graph-traversal-patterns
description: >
  Use when a problem is secretly a graph — reachability, shortest paths,
  ordering with dependencies, connected groups — choosing between BFS, DFS,
  topological sort, and union-find, with templates. Triggers: "shortest path",
  "dependency order", "connected components", "cycle detection", "islands",
  "prerequisite/build order", "is X reachable from Y".
---

# Graph Traversal Patterns

## When to use this skill
- The problem has entities + relationships and asks about connectivity, ordering, distance, or groups — including the disguised cases: grids, prerequisite lists, state spaces ("minimum moves to reach..."), reference chains.
- NOT for weighted shortest paths with arbitrary weights — that's Dijkstra/Bellman-Ford territory; this skill covers the unweighted/uniform core that solves 90% of practical cases (and names when to escalate).

## Prerequisites
- `data-structure-selection` — adjacency list as dict-of-lists is the default representation; matrices only for dense graphs or when the grid IS the matrix.

## Workflow

1. **First, see the graph.** Nodes = the states/entities; edges = the allowed moves/relations. Grids: cells + 4/8-neighbors. Prerequisites: courses + directed depends-on edges. Word ladders: words + one-letter-apart edges. String transformations, lock combinations, git commits — the modeling step is where the problem is actually solved; the traversal is clerical after that.

2. **Choose the algorithm by the question asked:**
   - **Shortest path / minimum steps (unweighted)** → **BFS**, no exceptions. BFS visits in distance order; DFS finds *a* path, not the shortest — using DFS for "minimum moves" is the classic wrong answer.
   - **Reachability / exhaustive exploration / path-with-backtracking** → **DFS** (recursive or explicit stack).
   - **Valid ordering under dependencies / "can it be done at all"** → **topological sort** (Kahn's BFS-on-indegrees, which detects cycles for free: leftover nodes = cycle).
   - **Dynamic grouping / "are these connected" under incremental unions** → **union-find** (no traversal at all — the right tool when edges arrive over time or queries interleave with merges).
   - Weights appear → Dijkstra (uniform-positive: BFS stops being valid the moment edges cost differently); 0/1 weights → 0-1 BFS with a deque; negative weights → Bellman-Ford. Name the escalation, don't force BFS.

3. **BFS template with the two invariants that prevent the common bugs:**
   ```python
   from collections import deque
   dist = {start: 0}                 # visited-check AT ENQUEUE time
   q = deque([start])
   while q:
       node = q.popleft()
       for nxt in neighbors(node):
           if nxt not in dist:       # mark when enqueued, NOT when popped
               dist[nxt] = dist[node] + 1
               q.append(nxt)
   ```
   Mark-on-enqueue (marking on pop lets the same node enter the queue many times — exponential blowup on dense graphs); distance stored with the node (parallel `level` counters drift). Multi-source BFS ("nearest hospital for every cell"): seed the queue with ALL sources at distance 0 — one pass, not one BFS per source.

4. **DFS: recursive for clarity, iterative for depth.** Recursion depth = longest path — a 10⁶-node line graph overflows the stack (`complexity-analysis` step 6); convert to an explicit stack when input size threatens the limit. For cycle detection in *directed* graphs, three colors (white/gray/black): a gray→gray edge is a cycle; the two-state visited set gives false positives on diamonds. Undirected cycle detection: visited + "isn't my parent," or union-find.

5. **Topological sort — Kahn's, because it answers both questions:**
   ```python
   indeg = count_indegrees(g); q = deque(n for n in g if indeg[n] == 0)
   order = []
   while q:
       n = q.popleft(); order.append(n)
       for m in g[n]:
           indeg[m] -= 1
           if indeg[m] == 0: q.append(m)
   if len(order) < len(g): raise CycleError(...)   # the free cycle check
   ```
   Real-world uses beyond interviews: build systems, migration ordering, `ci-pipeline-design` stage graphs, resolving `medallion-architecture-design` model dependencies. Need "which nodes ARE the cycle" for the error message? The leftovers with indeg > 0 contain it.

6. **Union-find in eight lines, with both optimizations** (path compression + union by size — together: effectively O(1) per op):
   ```python
   parent = list(range(n)); size = [1]*n
   def find(x):
       while parent[x] != x:
           parent[x] = parent[parent[x]]; x = parent[x]
       return x
   def union(a, b):
       ra, rb = find(a), find(b)
       if ra == rb: return False
       if size[ra] < size[rb]: ra, rb = rb, ra
       parent[rb] = ra; size[ra] += size[rb]; return True
   ```
   `union` returning False = "already connected" = the cycle/redundant-edge detector.

7. **Close with the graph-shaped edge cases:** disconnected graphs (loop over all nodes as potential starts — single-start traversal silently ignores other components), self-loops and parallel edges (does the model allow them?), empty graph / single node, and the grid-problem duo: bounds checks and the visited-marking that must happen before the recursive call (or the same cell enqueues four times from its four neighbors).

## Common pitfalls
- DFS for shortest path (step 2's table exists for this) — returns the first path found, tests pass on small inputs where first ≈ shortest, production finds the counterexample.
- Mark-on-pop BFS: correct answers, exponential queue. The bug that only shows at scale.
- Recursion-depth overflow on the "simple DFS" — grids of 1000×1000 have million-long snake paths.
- Treating an undirected problem's edges as directed (adding one direction of each edge) — half the graph unreachable, mysteriously.
- Rebuilding connectivity from scratch per query ("are A and B connected?" → full BFS each time) when union-find amortizes the whole query stream to near-linear.
- Missing the modeling move entirely: 40 lines of bespoke state-tracking for what is a 6-line BFS over a state graph — if the code has "try all possibilities" plus "avoid revisiting," it's a traversal wearing a trenchcoat.

## Example
Real task: "given 40k microservice dependency edges, produce a deploy order, and if impossible, name the offending services." Model: services = nodes, depends-on = directed edges (step 1: direction chosen deliberately — deploy dependencies *first*, so edges point dependency→dependent for Kahn's). Kahn's ran: 39,200 services ordered, 14 left over → cycle exists; the leftover subgraph trimmed to its core (repeatedly strip indeg-0/outdeg-0) exposed a 3-service circular dependency that two teams each believed was one-directional. Deliverable: the order file + the 3-service cycle named in the error. Runtime: 0.3s. The near-miss: the first draft used DFS post-order toposort without cycle handling — it would have produced a *plausible-looking wrong order* on the cyclic input; Kahn's leftover-check made the cycle a loud failure instead (`error-handling-strategy`'s fail-fast, algorithm edition).

## Related skills
- `data-structure-selection` — adjacency representations and union-find's home row.
- `complexity-analysis` — O(V+E) arguments and the recursion-depth space costs.
- `dynamic-programming-derivation` — DAG problems where the traversal carries values (longest path in DAG = DP over the toposort).
