---
name: graph-traversal-patterns
description: >
  Use when a problem is secretly a graph — reachability, shortest paths,
  ordering with dependencies, connected groups — choosing between BFS,
  DFS, topological sort, and union-find, with templates. Triggers:
  "shortest path", "dependency order", "connected components", "cycle
  detection", "islands", "prerequisite/build order", "is X reachable from
  Y". NOT for weighted shortest paths with arbitrary weights — Dijkstra/
  Bellman-Ford territory; this skill covers the unweighted core and names
  the escalation.
---

# Graph Traversal Patterns

## Overview

The modeling step — seeing that the grid, the prerequisite list, or the state space IS a graph — solves the problem; the traversal is clerical after that. The algorithm follows from the question: distance wants BFS, ordering wants Kahn's, dynamic grouping wants union-find, and using DFS for "minimum steps" is the genre's classic wrong answer.

## When to Use

- Entities + relationships with questions about connectivity, ordering, distance, or groups — including disguised cases: grids, dependency lists, state spaces ("minimum moves to..."), reference chains.

**When NOT to use:**
- Arbitrary-weight shortest paths — Dijkstra (positive), 0-1 BFS (0/1 weights), Bellman-Ford (negative). Name the escalation; don't force BFS.

## Prerequisites

- `data-structure-selection` — adjacency list as dict-of-lists is the default representation; matrices only for dense graphs.

## The Workflow

1. **First, see the graph.** Nodes = the states/entities; edges = the allowed moves/relations. Grids: cells + 4/8-neighbors. Prerequisites: directed depends-on edges. Word ladders: words one letter apart. If the code has "try all possibilities" plus "avoid revisiting," it's a traversal wearing a trenchcoat.

2. **Choose the algorithm by the question asked:**

   ```
   Question:
     minimum steps / shortest (unweighted)  → BFS, no exceptions
     reachability / explore / backtrack     → DFS
     valid order under dependencies         → topological sort (Kahn's)
     dynamic grouping / incremental unions  → union-find
     weights appear                         → escalate (Dijkstra family)
   ```

   DFS finds *a* path, not the shortest — "minimum moves via DFS" passes small tests where first ≈ shortest and fails in production.

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
   Mark-on-enqueue (marking on pop lets the same node enter the queue many times — exponential blowup on dense graphs); distance stored with the node (parallel level counters drift). Multi-source BFS ("nearest X for every cell"): seed the queue with ALL sources at distance 0 — one pass, not one BFS per source.

4. **DFS: recursive for clarity, iterative for depth.** Recursion depth = longest path — a 10⁶-node line graph overflows the stack (`complexity-analysis` step 6); convert to an explicit stack when input size threatens the limit (grids of 1000×1000 have million-long snake paths). Directed cycle detection: three colors (white/gray/black) — a gray→gray edge is a cycle; two-state visited gives false positives on diamonds. Undirected: visited + "isn't my parent," or union-find.

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
   Real-world uses: build systems, migration ordering, CI stage graphs, dbt model dependencies. Need "which nodes ARE the cycle"? The leftovers with indeg > 0 contain it.

6. **Union-find in eight lines, with both optimizations** (path compression + union by size — together effectively O(1) per op):
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
   `union` returning False = "already connected" = the cycle/redundant-edge detector. Use it over repeated BFS when queries interleave with merges.

7. **Close with the graph-shaped edge cases:** disconnected graphs (loop over all nodes as starts — single-start traversal silently ignores other components), self-loops and parallel edges (does the model allow them?), empty/single-node, and the grid duo: bounds checks and visited-marking BEFORE the recursive call (or the same cell enqueues four times). Undirected problems need BOTH edge directions added — half the graph unreachable, mysteriously, otherwise.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "DFS found a path — that's the answer to 'minimum moves'" | DFS finds the FIRST path, which small tests confuse with the shortest. The step-2 table has 'no exceptions' on BFS-for-distance because this wrong answer passes review constantly. |
| "Mark-on-pop, mark-on-enqueue — same thing" | Mark-on-pop lets a node enter the queue once per in-edge: correct answers, exponential queue on dense graphs. The bug that only shows at scale. |
| "It's not a graph problem — it's just state tracking" | 'Try all possibilities' + 'avoid revisiting' IS a traversal; 40 lines of bespoke state machinery re-derive a 6-line BFS, with more bugs. The modeling step is the solution. |
| "Recursion is fine — the input won't be that deep" | 10⁶-node line graphs and 1000×1000 grid snakes overflow at depths the happy-path never hits. Depth is input-controlled; the explicit stack is insurance priced at five lines. |
| "Two-state visited detects directed cycles" | It false-positives on diamonds (two paths to the same node ≠ cycle). Three colors or the 'cycle' you report is just convergence. |
| "Re-run BFS per connectivity query — simple" | With merges interleaving queries, union-find amortizes the stream to near-linear; per-query BFS is the accidental O(VE). The op-mix decides (data-structure-selection's rule). |

## Red Flags

- "Shortest"/"minimum" in the problem, DFS in the solution.
- BFS with visited-marking at pop time.
- Toposort via DFS post-order with no cycle handling — plausible wrong orders on cyclic input.
- Grid DFS with recursion and no depth consideration.
- One edge direction added for an undirected problem.
- Connectivity queries re-traversing per call while unions happen between them.

## Verification

- [ ] The graph model stated: what are nodes, what are edges, directed or not — one line.
- [ ] Algorithm choice justified against the question (the step-2 table row named).
- [ ] BFS solutions: enqueue-time marking verified — code inspected.
- [ ] Cycle-detection method matches directedness (3-color vs parent-skip) — noted.
- [ ] Disconnection handled: multi-start loop present or single-component assumption justified.
- [ ] Edge-case battery run: empty, single node, disconnected, self-loop — tests listed.

## Example

Real task: "given 40k microservice dependency edges, produce a deploy order, and if impossible, name the offending services." Model: services = nodes, depends-on = directed edges (direction chosen deliberately — dependencies deploy first). Kahn's ran: 39,200 services ordered, 14 left over → cycle exists; the leftover subgraph trimmed to its core exposed a 3-service circular dependency that two teams each believed was one-directional. Deliverable: the order file + the 3-service cycle named. Runtime: 0.3s. The near-miss: the first draft used DFS post-order toposort without cycle handling — it would have produced a *plausible-looking wrong order* on the cyclic input; Kahn's leftover-check made the cycle a loud failure instead.

## Related skills

- `data-structure-selection` — adjacency representations and union-find's home row.
- `complexity-analysis` — O(V+E) arguments and recursion-depth space costs.
- `dynamic-programming-derivation` — DAG problems where the traversal carries values.
