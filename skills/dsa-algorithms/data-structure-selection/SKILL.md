---
name: data-structure-selection
description: >
  Use when choosing the data structure for an access pattern — the decision
  table from operations-needed to structure, with the real-world costs the
  textbook table omits. Triggers: "what data structure should I use",
  "dict or list here", "need fast lookup and ordering", "priority queue or
  sorted list", "how should I store this".
---

# Data Structure Selection

## When to use this skill
- Code needs a container and the choice isn't obviously "just a list."
- A profiler/review flagged a structure misuse (the O(n) membership check).
- NOT for database/storage-layer choices — same instincts, different skill (`sql-query-optimization` for indexes, which ARE this table at disk scale).

## Prerequisites
- The operation mix, written down: which operations, roughly how often, at what n. "Insert-heavy, occasional lookup, n≈10⁵" decides more than any benchmark.

## Workflow

1. **List the operations the code actually performs** — not the data's "natural shape." The same "list of users" wants: array if iterated; dict-by-id if looked up; heap if "next expiring"; deque if FIFO. Structure follows access, not nouns.

2. **Match against the core table** (average cases; language names vary):

   | Need | Structure | Cost | The catch |
   |---|---|---|---|
   | Index access, iteration, append | dynamic array (`list`, `Vec`) | O(1) | insert/delete at front/middle O(n) |
   | Membership, lookup by key | hash map/set (`dict`, `HashMap`) | O(1) avg | no order†; hashing cost on huge keys; worst-case O(n) |
   | FIFO / both-ends ops | deque / ring buffer | O(1) | mid-access O(n) |
   | LIFO | array as stack | O(1) | — |
   | Repeated min/max ("next by priority") | binary heap | O(log n) push/pop, O(1) peek | no efficient arbitrary lookup/delete‡ |
   | Ordered iteration + insert + range queries | balanced BST / skip list (`SortedList`, `TreeMap`, `BTreeMap`) | O(log n) | the structure Python/JS don't ship natively — know your library |
   | Prefix/string-key queries | trie | O(key length) | memory-hungry; usually a dict is fine until it isn't (`string-algorithm-toolkit`) |
   | Connectivity / grouping | union-find | ~O(1) amortized | union/find only — no un-merge |
   | Frequency counting | hash map of counts (`Counter`) | O(1) per event | it's just a dict; top-k needs a heap on top (`heap-and-priority-patterns`) |

   †Python dicts preserve *insertion* order — that's not *sorted* order; the confusion causes real bugs.
   ‡"Delete arbitrary item from heap" → lazy deletion (tombstone set, skip on pop) beats re-heapifying.

3. **Composite structures for composite needs — the standard combos:** LRU cache = hash map + doubly-linked list (`OrderedDict`/`LinkedHashMap` pre-builds it); "lookup by id AND iterate by score" = dict + sorted structure maintained together (wrap the pair in a class so the two can't drift); graph = dict-of-lists adjacency (`graph-traversal-patterns`); interval/range aggregation at scale = the point where you've outgrown this table (segment trees / interval trees — verify you truly need them first).

4. **Apply the reality corrections the textbook omits:** below n≈10³, arrays beat everything via cache locality (a linear scan of 50 items outruns a tree, and often the hash) — don't build machinery for small n (`complexity-analysis` step 7); memory overhead is real (a Python dict of small ints costs ~10× the array; at n=10⁸ the "fast" structure may not fit — arrays/numpy win by fitting in RAM); hash costs scale with key size (hashing 1KB strings per lookup isn't O(1) in any sense that matters — intern or pre-hash).

5. **Check the concurrency and mutation constraints:** mutated-while-iterated (most languages: error or corruption — collect-then-mutate, or use the structure's safe idiom); shared across threads (per-language answer: locks, `ConcurrentHashMap`, immutable + replace — the default structure is almost never thread-safe); persistence/undo needs favor immutable/persistent structures.

6. **Default simple, upgrade on evidence.** The honest priority: array/list → dict/set the moment lookup or membership appears → everything else only when the operation mix demands it. Exotic structures carry a bug tax and a reader tax; a heap earns its place when "repeatedly need the min" is *in the requirements*, not in the imagination (`# ponytail`-adjacent instinct, but the upgrade trigger is the profiler or the written op-mix, not aesthetics).

## Common pitfalls
- List membership in a loop — the classic accidental O(n²) (`complexity-analysis` step 3). The fix is one word (`set`); noticing is the skill.
- Sorting the whole array every iteration to get the min (O(n log n) per step) where a heap does O(log n) — or, inverted, building a heap to find the min *once* (O(n) scan wins).
- `list.insert(0, ...)` / `shift()` as a queue: O(n) per op. Deque exists.
- Choosing by noun ("it's a ranking → sorted structure") when the ops are "batch-compute weekly, read top 10" — an array sorted once. Access pattern, not noun.
- Two parallel structures (dict + list of the same items) drifting out of sync because they're updated in 9 places — encapsulate the invariant (step 3).
- Premature exotica: the interval tree for 40 intervals, the trie for 200 words. n decides; small n forgives everything and punishes cleverness with bugs.

## Example
Task: "match incoming orders against the best open quote; quotes arrive, expire, and get cancelled; ~50k live quotes." Op mix written down: insert quote (frequent), pop best-price (frequent), cancel by id (frequent), expire by time (periodic). Best-price → heap keyed on price; cancel-by-id against a heap → the ‡ catch: lazy deletion — a `cancelled: set[QuoteId]`, popped quotes checked against it and skipped. Expiry → second heap keyed on expiry time, same tombstone set. Wrapped as one `QuoteBook` class holding both heaps + the set + a by-id dict, so the four structures can't drift. Considered and rejected: a sorted container (O(log n) for everything including the cancels — cleaner, but the language stdlib lacked one and the two-heap + tombstone pattern is boring and testable). Load test at 10× volume: pop p99 = 40µs. The alternative first draft — re-sorting a list per match — had been measured at 80ms per op: a 2000× difference from choosing by access pattern.

## Related skills
- `complexity-analysis` — verifying the chosen structure's bound survives the input.
- `heap-and-priority-patterns` — the heap rows of the table, expanded.
- `graph-traversal-patterns` — adjacency representations and their traversals.
- `string-algorithm-toolkit` — the trie row, expanded.
