---
name: edge-case-enumeration
description: >
  Use when you need to systematically find the inputs that break code —
  boundaries, empties, nulls, encoding, time, concurrency — instead of
  guessing. Works for writing tests or reviewing them. Triggers: "what edge
  cases am I missing", "test cases for this function", "what could break
  this", "boundary conditions", "did I cover everything".
---

# Edge Case Enumeration

## When to use this skill
- Writing tests and wanting confidence the input space is covered.
- Reviewing a PR's tests for gaps.
- NOT a substitute for property-based testing on truly large input spaces — see `property-based-testing` when invariants exist.

## Prerequisites
- The function/endpoint signature and what each input means (units, valid ranges).

## Workflow

Walk each input through the checklist for its type, then the cross-cutting categories. Write down every case that's *reachable*; strike ones validation upstream provably blocks.

1. **Numbers:** zero, negative, boundary value and boundary±1 (off-by-one lives here), max/min of the type, float specials (NaN, ±Inf, -0.0), values causing overflow in *intermediate* computation (`(a+b)/2`), precision traps (0.1+0.2), and huge-but-valid (1e15 cents).

2. **Strings:** empty, single char, whitespace-only, leading/trailing whitespace, very long (past any column/buffer limit), Unicode beyond ASCII (é, 中, emoji incl. multi-codepoint 👨‍👩‍👧), combining characters vs precomposed (é as one codepoint vs e+◌́), null byte, newline/tab embedded, case variants if compared, and strings that are *syntax* somewhere downstream: `'; --`, `<script>`, `{{}}`, `%s`, a filename like `../../etc/passwd`.

3. **Collections:** empty, one element, exactly the page/batch-size boundary and ±1, duplicates, all-identical elements, already-sorted and reverse-sorted (perf + logic), nested empties (`[[]]`), and max realistic size.

4. **Null/absent — distinguish the three:** missing key, key present with null, key present with empty value. APIs and DBs treat these differently; tests should too.

5. **Time:** midnight, end of month (Jan 31 + 1 month = ?), Feb 29, Dec 31→Jan 1, DST spring-forward gap (2:30am doesn't exist) and fall-back overlap (1:30am happens twice), epoch 0, pre-1970, timezone-naive meets timezone-aware, and "now" moving between two reads in the same function.

6. **State & sequence:** operation on uninitialized/closed/already-done state (cancel a cancelled order), repeated identical calls (idempotency), operations out of expected order, and the first-ever call (empty table, cold cache).

7. **Concurrency (when shared state exists):** two identical requests simultaneously (double-submit), read-modify-write interleaving, and the row deleted between your check and your use (TOCTOU).

8. **Rank and cut.** Score each case: likelihood × damage. Test everything user-reachable with real damage; drop cases upstream types/validation genuinely make impossible — and add one test *proving* the validation blocks them.

## Common pitfalls
- Testing boundary but not boundary±1. `limit=100` passing says little; 99, 100, 101 is the actual off-by-one probe.
- Only happy-path Unicode ("José") while multi-codepoint emoji and combining chars are what actually break length checks and slicing.
- Treating unreachable-by-UI as unreachable — the API accepts what the form wouldn't send.
- Enumerating 200 cases and testing all of them equally. Without step 8's cut, the suite is slow and the valuable cases drown.
- Forgetting intermediate overflow: inputs individually valid, sum not.

## Example
`paginate(items, page, per_page)` — enumeration found: `page=0` (is it 1-indexed?), negative page, page past the end, `per_page=0` (division), empty items, items length exactly `per_page` (does page 2 exist as empty or 404?), and `per_page=10**9` (allocation). Eight tests; two found real bugs: `page=0` returned the last page via Python negative indexing, and length-equals-per_page produced a phantom empty page 2 that broke the mobile client's infinite scroll.

## Related skills
- `unit-test-design` — the structure these cases get written into.
- `property-based-testing` — generating cases from invariants instead of checklists.
- `input-validation-boundaries` — deciding where these cases get *blocked* rather than handled.
