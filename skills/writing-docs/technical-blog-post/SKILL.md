---
name: technical-blog-post
description: >
  Use when writing a technical blog post or engineering article — structuring
  hook, problem, journey, takeaway; cutting throat-clearing; making it worth
  a stranger's ten minutes. Triggers: "write a blog post about", "engineering
  blog draft", "turn this project into a post", "review my article",
  "postmortem writeup for the blog".
---

# Technical Blog Post

## When to use this skill
- Turning a project, incident, migration, or investigation into a public/internal article.
- Reviewing a draft that feels flat or unfocused.
- NOT for reference documentation (`api-documentation`) or the prose-polish pass itself (`natural-prose-editing` — run it after this skill settles the structure).

## Prerequisites
- The actual experience to write about, with its artifacts: numbers, graphs, failed attempts, code. Posts assembled from memory read like posts assembled from memory.

## Workflow

1. **Write the one-sentence takeaway first, and make it earn disagreement.** "What will a reader know or do differently after this?" — "We cut p99 latency 8× by removing a cache" is a post; "caching is important" is not. If the takeaway is something nobody would dispute, there's no post yet — find the surprising part (the cache REMOVAL is the hook precisely because it inverts the expected advice).

2. **Open with the payoff, not the biography.** First paragraph: the problem's stakes and the destination — "Our search p99 was 4 seconds. This post covers how it became 300ms, and why the fix was deleting our caching layer." Readers grant ~15 seconds before bouncing; the company-history and team-intro paragraphs spend them on nothing. (Context that matters gets woven in where it becomes relevant.)

3. **Structure as the journey, honestly including the dead ends.** The narrative spine that works for engineering posts: symptom → investigation → wrong turn(s) → the realization → the fix → the numbers → what transfers. The failed attempts are not filler — they're the most instructive part (readers are standing at the same forks) and the most trust-building (nobody believes the straight-line version). One or two dead ends, told briefly, each with why it seemed right and what disproved it.

4. **Show the evidence: numbers, graphs, code — each pulling weight.** Before/after metrics with axes and units (a graph with no y-axis scale is an illustration, not evidence); code snippets trimmed to the interesting delta (10 lines with the crucial part highlighted beats 80 lines of context — link the full version); the actual error message, the actual flame graph. Every artifact answers "how do you know?"

5. **Write for one named reader.** Pick them: "backend engineer who runs Postgres but hasn't touched logical replication." That choice settles every what-do-I-explain decision — you explain replication slots (they haven't touched them) but not SQL (they run Postgres). Posts pitched at everyone explain everything, bore everyone, and run 4,000 words.

6. **Cut the ritual sections:** the "in this post we will..." roadmap (just start), the recap conclusion that re-summarizes what the reader just read (end on the transferable lesson or the open question instead), and hedging preambles ("this may not apply to every situation..." — state the applicability conditions once, concretely). Target: most posts land at 800–1,500 words; past 2,000, either split it or you're padding.

7. **Run the two-pass review:** technical accuracy by someone who knows the domain (would they cringe? every simplification checked against "is this false or just simplified?"), then readability by someone matching the step-5 reader (where did they skim? that's where it drags). Then the `natural-prose-editing` pass for the sentence-level flatness, and title last — specific and honest ("How a 40-character regex took down our API" beats "Our Journey with Regular Expressions"), because the title is the only part most people will ever read.

## Common pitfalls
- Writing the journey in the order you lived it, including the three boring weeks — chronology is not structure; the narrative spine (step 3) compresses time ruthlessly around the turns.
- The humble-brag postmortem that sands off the actual mistake ("we identified an optimization opportunity" for "we shipped a config that took the site down") — the honest version is the one that gets read, shared, and believed.
- Evidence-free claims ("this dramatically improved performance") in a genre where the whole currency is receipts. Numbers or it didn't happen.
- Explaining your stack's basics to readers you defined as already knowing them — the step-5 contract violated, 800 words wasted.
- Code dumps: whole files pasted where the delta was 6 lines. The reader's eyes bounce off; the 6 lines with one highlight would have landed.
- Publishing the first draft's structure: draft 1 discovers what the post is about (the takeaway often emerges at the END of draft 1 — move it to the top and rebuild).

## Example
Raw material: a two-week incident investigation ending in "the cache was the bottleneck." Draft 1 was chronological: 2,600 words, takeaway buried in paragraph 19. Rework per the spine: takeaway extracted and promoted ("the cache we added for performance was costing us 3.7s of p99"); opener rewritten to stakes+destination; the two dead ends kept (blaming the DB — disproven by the trace screenshot included; blaming GC — disproven by the pause histogram) at a paragraph each; the realization scene (the flame graph showing lock contention *in the cache client*) given the post's only full-width image; before/after latency graph with marked deploy line; 60-line handler cut to the 8-line diff of the removal. Named reader: "service owner who has added a cache under pressure" — so cache-stampede basics explained in one clause, Go profiling not explained at all. Final: 1,340 words. Accuracy pass caught one false simplification ("Redis is single-threaded" → scoped to the command-execution path). It became the team's most-shared post, and the title that shipped — "The cache that cost us 3.7 seconds" — was rewrite #6.

## Related skills
- `natural-prose-editing` — the sentence-level pass after structure settles.
- `blameless-postmortem` — incident writeups this genre often grows from.
- `stakeholder-update` — the internal, compressed sibling.
- `readme-authoring` — the same show-the-output discipline, different door.
