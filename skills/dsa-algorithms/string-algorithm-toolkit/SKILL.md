---
name: string-algorithm-toolkit
description: >
  Use when string problems outgrow brute force — substring search, prefix
  queries, string hashing, and knowing when naive is genuinely fine.
  Triggers: "substring search at scale", "find all occurrences", "autocomplete
  prefix matching", "string matching too slow", "rolling hash", "trie or dict".
---

# String Algorithm Toolkit

## When to use this skill
- String matching/indexing where input size makes the naive approach a real cost — or where you suspect it might and want the threshold check.
- Choosing between dict, trie, hashing, and automaton approaches.
- NOT for regex authoring or parsing/grammars — different toolset; and NOT for fuzzy/semantic matching (that's edit distance DP or embeddings).

## Prerequisites
- Sizes written down: text length n, pattern length m, number of patterns k, alphabet. The whole skill is choosing by these numbers.

## Workflow

1. **Start from the do-nothing baseline: your language's built-in find is excellent.** `str.find` / `in` / `indexOf` are C-optimized, often SIMD-accelerated, typically implementing two-way/Boyer-Moore-class algorithms already. For one pattern in one text, the built-in wins below n ≈ 10⁷ almost regardless of theory. Escalate only on: many patterns, many queries against one text, adversarial input, or streaming.

2. **One pattern, many texts, or paranoia about worst cases → know what KMP buys.** Naive matching is O(nm) worst case but O(n) on random text; KMP guarantees O(n+m) with no backtracking on the text (streaming-safe). In practice: use the stdlib until profiling or adversarial input (pathological repetition — `"aaa...a"` patterns) proves otherwise; implement KMP's failure function only when you need its *byproduct* — the border array (longest proper prefix = suffix), which solves period-detection and "shortest palindrome"-family problems directly.

3. **Many patterns against one stream → Aho-Corasick.** k patterns matched simultaneously in O(n + total pattern length + matches): the content-filter / log-scanner / intrusion-detection shape (`pyahocorasick`, or trie + failure links hand-rolled). k separate `find` passes = O(kn) — the difference between 200 keyword scans per document and one.

4. **Many substring queries on one fixed text → rolling hash (or suffix structures at the extreme).** Polynomial rolling hash gives O(1) substring-equality after O(n) precompute:
   ```python
   # h[i] = hash of s[:i]; hash(s[l:r]) = (h[r] - h[l]*pow[r-l]) % M
   ```
   Rules that keep it honest: large prime modulus (or two moduli — collision probability squared), random base per run (defeats crafted collisions — hashes on adversarial input are a security surface, `secure-code-review` step 6's hash-flooding cousin), and **verify candidate matches by direct comparison** when a false positive costs correctness. Rabin-Karp = this + sliding. Suffix arrays/automata: the right tool for "longest repeated substring"/"count distinct substrings" — reach for a library; hand-rolling one under deadline is how weekends die.

5. **Prefix-shaped workloads → trie, but only past the dict threshold.** Autocomplete, prefix counts, longest-common-prefix sets. Below ~10⁴ keys, `bisect` on a sorted list or a dict of prefixes is simpler and often faster (cache locality; tries chase pointers). The trie earns its memory when: prefix *enumeration* is the hot path, keys share long prefixes heavily, or per-node augmentation (counts, best-completion) is needed. Compressed variants (radix tree) when memory bites — `data-structure-selection` step 4's overhead math applies double to tries.

6. **Normalize before matching, or match wrong:** Unicode NFC/NFD (é as one codepoint vs two — `edge-case-enumeration` step 2's combining characters break naive equality), case-folding (`casefold()`, not `lower()`, for real i18n), and decide byte-vs-codepoint-vs-grapheme semantics explicitly (emoji and combining marks make `len()` a lie in three different ways). Every matching bug report that "only happens for some users" starts here.

7. **Close with the honesty check: state why the chosen tier is needed.** One sentence in the PR/comment: "Aho-Corasick: 3k patterns × 1M docs/day makes per-pattern find O(kn) ≈ 10¹¹" — or, equally valid, "naive scan: n ≤ 10⁴, runs hourly; stdlib `in` suffices." The toolkit's most-used tool should be the baseline; escalation without the sizes written down is résumé-driven engineering.

## Common pitfalls
- Hand-implementing Boyer-Moore/KMP to beat a stdlib that already implements better — slower AND buggier. Measure first.
- Rolling hash treated as equality: two matches with equal hashes merged without verification; the one-in-a-billion collision arrives at one-billion scale, on schedule.
- Fixed hash base+modulus in anything user-facing — crafted collision inputs degrade Rabin-Karp to O(nm) or corrupt dedup results (adversarial input is a *when*, not an *if*, on public surfaces).
- Trie for 500 dictionary words behind an API that gets 10 QPS — pointer-chasing machinery where `startswith` in a loop was invisible in the profile.
- Matching raw bytes while users type composed/decomposed Unicode — "search can't find café" bugs, filed forever, one normalization call away.
- O(n²) substring generation (`all substrings, then dedupe`) sneaking in before the clever matcher — the surrounding code's complexity swamping the algorithm's (`complexity-analysis` step 3's hidden costs).

## Example
Real task: compliance scanner — 4,200 restricted phrases against ~800k support messages/day, currently 40 min/day via per-phrase `in` loops and growing linearly with the phrase list. Sizes: k=4200, n≈10⁹ chars/day → O(kn) ≈ 4×10¹² — the escalation is justified on paper before any code (step 7 in reverse). Aho-Corasick automaton built once at startup (0.8s, 60MB), stream scanned in one pass: 92 seconds/day, output = (message, phrase, offset) triples. Normalization pass added after week one: NFKC + casefold before both automaton build and scan — the missed matches in the pilot were decomposed accents from mobile keyboards, exactly the step-6 classic. Phrase list doubled the next quarter; scan time moved 92s → 96s. The rejected alternative (regex alternation of 4200 phrases) had been tried previously: the compiled pattern hit catastrophic size and the scan ran hours.

## Related skills
- `data-structure-selection` — the trie-vs-dict threshold logic.
- `complexity-analysis` — the sizes-first discipline this skill runs on.
- `edge-case-enumeration` — the Unicode cases that break matching silently.
- `two-pointer-sliding-window` — character-window problems that need no toolkit at all.
