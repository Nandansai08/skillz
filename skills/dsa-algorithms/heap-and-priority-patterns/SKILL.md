---
name: heap-and-priority-patterns
description: >
  Use when a problem repeatedly needs the current min/max — top-k selection,
  merging k sorted streams, scheduling by priority, running medians —
  with heap templates and the size-k trick. Triggers: "top k", "k largest",
  "merge k sorted", "task scheduling by priority", "running median",
  "kth smallest", "process in priority order".
---

# Heap & Priority Patterns

## When to use this skill
- "Repeatedly extract the best" appears in the requirements: top-k, k-way merge, priority scheduling, sliding medians.
- NOT for one-shot min/max (a scan), full sorting (just sort), or lookup-heavy mixes (see `data-structure-selection`'s table — heaps don't do arbitrary lookup).

## Prerequisites
- The op-mix habit from `data-structure-selection`: heaps pay off when insertions interleave with best-extractions; batch-then-sort beats a heap when all data arrives before any query.

## Workflow

1. **Know the two library gotchas before any pattern:** most stdlib heaps are min-heaps only (`heapq`) — max-heap = negate keys (`heappush(h, -x)`) and negate on read; and heap entries with equal keys need a tiebreaker when payloads aren't comparable — the standard entry is `(key, seq, payload)` with an incrementing `seq`:
   ```python
   heappush(h, (priority, next(counter), task))   # counter breaks ties, task never compared
   ```

2. **Top-k: the inverted-heap-of-size-k trick — the pattern's crown jewel.** K largest items from n (or a stream): keep a MIN-heap of size k; each new item compares against the heap's minimum (the weakest of the current top-k) and replaces it if larger:
   ```python
   h = nums[:k]; heapq.heapify(h)
   for x in nums[k:]:
       if x > h[0]: heapq.heapreplace(h, x)
   ```
   O(n log k) time, O(k) space — works on streams too big to hold, which O(n log n) sorting can't. The inversion (min-heap for k-largest) is the part everyone gets backwards once; the heap holds the *candidates*, so its root is the eviction boundary. Shortcuts worth knowing: `heapq.nlargest(k, ...)` wraps this; quickselect gives O(n) average for one-shot arrays when streaming isn't needed.

3. **K-way merge: heap of (head, source) pairs.** Merging k sorted lists/streams/files: seed with each source's first element, pop the global min, push that source's next:
   ```python
   h = [(lst[0], i, 0) for i, lst in enumerate(lists) if lst]
   heapq.heapify(h)
   while h:
       val, src, idx = heappop(h); out.append(val)
       if idx + 1 < len(lists[src]): heappush(h, (lists[src][idx+1], src, idx+1))
   ```
   O(N log k) for N total items. This IS external merge sort's core, log-structured storage compaction, and `merge k sorted event streams by timestamp` in every pipeline — the most production-relevant heap pattern (`heapq.merge` packages it for iterables).

4. **Scheduling: heap keyed on next-eligible-time or priority.** Task queues, timer wheels' little sibling, "CPU scheduling" problems: push `(ready_time, seq, task)`, pop when `now >= ready_time`; two-heap variants (waiting-heap by ready-time + eligible-heap by priority) handle "highest priority among the currently-eligible" — the shape of most real schedulers and of interval problems like meeting-rooms (min-heap of end-times; heap size = rooms in use).

5. **Running median: two balanced heaps.** Max-heap of the lower half + min-heap of the upper half, rebalanced to size difference ≤ 1; median = a root (or their mean). The invariant to maintain: every lower-heap element ≤ every upper-heap element — push to one side, then rebalance by moving a root. Generalizes to running percentiles (size ratio p : 1−p) — the streaming-latency-p99 estimator's simple cousin.

6. **Arbitrary deletion/update: lazy tombstones, not surgery.** Heaps can't delete by value efficiently; the production answer is the tombstone set — mark deleted, discard on pop (`data-structure-selection`'s QuoteBook example). For decrease-key patterns (Dijkstra), push the updated entry and tombstone-skip stale pops (`if dist_seen < dist_in_entry: continue`) — simpler than indexed heaps and usually as fast. Keep the tombstone set bounded (drain it on pops) or the "deleted" majority bloats the heap.

7. **State the complexity honestly and check the alternative:** heapify is O(n) (not n log n — building beats n pushes); n pushes+pops = O(n log n) ≈ a sort — so if everything is pushed before anything is popped, sorting is simpler and cache-friendlier. The heap earns its keep on *interleaved* streams and *k ≪ n* selections; say which one applies or use the sort.

## Common pitfalls
- Min/max inversion in top-k (max-heap of size k for k-largest keeps the k *smallest* seen — plausible outputs on sorted-ish test data, wrong on real data).
- Comparing payloads: `heappush(h, (priority, task_dict))` crashes on the first tie when dicts can't compare — the seq-counter entry (step 1) is not optional hygiene, it's a latent prod crash.
- Mutating an element's key while it sits in the heap — silently breaks the invariant; every subsequent pop is untrustworthy. Tombstone + re-push (step 6), never in-place edits.
- Heap for "find the max once" (O(n) heapify to save an O(n) scan) or heap for fully-batched data (a sort with extra steps) — step 7's alternative check.
- Sliding-window maximum solved with a heap + tombstones (O(n log n), works) when the monotonic deque does O(n) — fine to ship the heap, but know the upgrade exists.
- Unbounded tombstone sets under heavy churn — the heap becomes 90% garbage and every pop pays log(garbage).

## Example
Real task: alert-storm dampener — from a stream of ~2M events/hour, maintain "the 50 highest-severity unresolved alerts" for the on-call dashboard, where alerts also *resolve* (removal) and *escalate* (key change). Design: min-heap of size ~50 won't work alone (removals + updates) → full min-heap keyed on `(-severity, seq)` with a tombstone set for resolved/stale entries, drained on pop; dashboard reads pop-and-peek the top 50 into a snapshot every 5s, re-pushing live entries. Escalations: push the new entry, tombstone the old `(alert_id, version)`. Tombstone bound: version counter per alert makes staleness checkable in O(1), set pruned on read. Load test at 10× storm volume: snapshot p99 = 4ms. The first draft had used `(priority, alert_dict)` entries — crashed on tied severities within the hour, exactly per step 1's warning; the seq counter fixed it in one line.

## Related skills
- `data-structure-selection` — when the answer is NOT a heap.
- `two-pointer-sliding-window` — window problems where a monotonic deque supersedes the heap.
- `graph-traversal-patterns` — Dijkstra as scheduling-by-distance with lazy deletion.
- `complexity-analysis` — the heapify-is-O(n) class of bound facts.
