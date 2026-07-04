---
name: technical-blog-post
description: >
  Use when writing a technical blog post or engineering article —
  structuring hook, problem, journey, takeaway; cutting throat-clearing;
  making it worth a stranger's ten minutes. Triggers: "write a blog post
  about", "engineering blog draft", "turn this project into a post",
  "review my article", "postmortem writeup for the blog". NOT for
  reference documentation (see api-documentation) or the sentence-level
  polish pass (see natural-prose-editing — run it after structure
  settles).
---

# Technical Blog Post

## Overview

A post is a takeaway a reader could dispute, delivered through an honest journey with receipts. Chronology is not structure, evidence is the genre's currency, and the dead ends — told briefly — are the most instructive and trust-building material you have.

## When to Use

- Turning a project, incident, migration, or investigation into a public/internal article.
- Reviewing a draft that feels flat or unfocused.

**When NOT to use:**
- Reference docs — `api-documentation`.
- Sentence-level de-flattening — `natural-prose-editing`, after this skill settles structure.

## Prerequisites

- The actual experience to write about, with its artifacts: numbers, graphs, failed attempts, code. Posts assembled from memory read like posts assembled from memory.

## The Workflow

1. **Write the one-sentence takeaway first, and make it earn disagreement.** "What will a reader know or do differently?" — "We cut p99 latency 8× by removing a cache" is a post; "caching is important" is not. If nobody would dispute the takeaway, there's no post yet — find the surprising part.

2. **Open with the payoff, not the biography.** First paragraph: stakes and destination — "Our search p99 was 4 seconds. This post covers how it became 300ms, and why the fix was deleting our caching layer." Readers grant ~15 seconds; company-history paragraphs spend them on nothing.

3. **Structure as the journey, honestly including the dead ends.** The spine: symptom → investigation → wrong turn(s) → realization → fix → numbers → what transfers. The failed attempts are not filler — readers stand at the same forks, and nobody believes the straight-line version. One or two dead ends, briefly, each with why it seemed right and what disproved it.

4. **Show the evidence: numbers, graphs, code — each pulling weight.** Before/after metrics with axes and units (a graph with no y-axis is an illustration, not evidence); code trimmed to the interesting delta (10 lines with the crucial part highlighted beats 80 of context); the actual error, the actual flame graph. Every artifact answers "how do you know?"

5. **Write for one named reader.** "Backend engineer who runs Postgres but hasn't touched logical replication" — that choice settles every explain-or-assume decision. Posts pitched at everyone explain everything, bore everyone, and run 4,000 words.

6. **Cut the ritual sections:** the "in this post we will..." roadmap (just start), the recap conclusion (end on the transferable lesson or the open question), hedging preambles (state applicability conditions once, concretely). Target 800–1,500 words; past 2,000, split or you're padding.

7. **Run the two-pass review, then polish and title:** technical accuracy by someone who knows the domain (every simplification checked against "is this false or just simplified?"), readability by someone matching the step-5 reader (where they skim is where it drags). Then `natural-prose-editing`, and the title last — specific and honest ("How a 40-character regex took down our API"), because the title is the only part most people will ever read.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "I'll write it in the order it happened — authentic" | Chronology includes the three boring weeks and buries the takeaway at paragraph 19 (the example's draft 1, exactly). The spine compresses time around the turns; authenticity lives in the dead ends, not the calendar. |
| "Cut the failed attempts — they make us look bad" | The dead ends are the most instructive part (readers face the same forks) and the most credible (nobody believes straight lines). Sanding them off produces the humble-brag nobody shares. |
| "Everyone knows the performance improved — numbers are fussy" | Evidence is the genre's currency; 'dramatically improved' is a claim, '4s → 300ms, graph attached' is a post. Receipts or it reads as marketing. |
| "Write for everyone — wider audience" | Everyone-posts explain everything and land with no one. The named reader converts each explain-or-skip decision from debate into lookup. |
| "The full 80-line handler gives important context" | Eyes bounce off code walls; the 8-line delta with one highlight lands. Link the full version for the three readers who want it. |
| "First draft's structure is fine — just needs polish" | Draft 1 DISCOVERS the takeaway, usually at its end. Publishing that structure buries the lede by construction; move it to the top and rebuild — then polish. |

## Red Flags

- The takeaway appearing for the first time in the final third.
- An opening paragraph about the company or team.
- Zero dead ends in a debugging/migration story.
- Graphs without axes; claims without numbers.
- "In this post we will..." / a conclusion that re-summarizes.
- 3,000+ words with no named reader anywhere in the process.

## Verification

- [ ] The disputable takeaway written before drafting — quoted in the draft doc.
- [ ] Opening paragraph carries stakes + destination — no biography.
- [ ] ≥1 dead end included with its why-it-seemed-right and its disproof.
- [ ] Every quantitative claim has its artifact (graph with axes, output, diff) — inline.
- [ ] Named reader stated; explain/assume decisions consistent with them.
- [ ] Both review passes done (accuracy + target-reader readability) — reviewers named; word count in range.

## Example

Raw material: a two-week incident investigation ending in "the cache was the bottleneck." Draft 1 was chronological: 2,600 words, takeaway buried in paragraph 19. Rework per the spine: takeaway extracted and promoted ("the cache we added for performance was costing us 3.7s of p99"); opener rewritten to stakes+destination; the two dead ends kept (blaming the DB — disproven by the trace screenshot included; blaming GC — disproven by the pause histogram) at a paragraph each; the realization scene (the flame graph showing lock contention *in the cache client*) given the post's only full-width image; before/after latency graph with marked deploy line; 60-line handler cut to the 8-line diff. Named reader: "service owner who has added a cache under pressure" — cache-stampede basics in one clause, Go profiling not explained at all. Final: 1,340 words. Accuracy pass caught one false simplification ("Redis is single-threaded" → scoped to the command-execution path). It became the team's most-shared post; the title that shipped — "The cache that cost us 3.7 seconds" — was rewrite #6.

## Related skills

- `natural-prose-editing` — the sentence-level pass after structure settles.
- `blameless-postmortem` — incident writeups this genre often grows from.
- `stakeholder-update` — the internal, compressed sibling.
- `readme-authoring` — the same show-the-output discipline, different door.
