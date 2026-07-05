---
name: literature-review
description: >
  Use when surveying what's known on a topic — systematic search strategy,
  source quality tiers, synthesis matrices, knowing when to stop reading.
  Triggers: "literature review on", "what's the state of the art",
  "survey the research on", "what's known about", "before we reinvent
  this, what exists". NOT for evaluating a specific tool/library for
  adoption (see technology-evaluation) — this skill maps a knowledge
  landscape.
---

# Literature Review

## Overview

A review's deliverable is claims with evidence weights, not proof-of-reading: the synthesis matrix converts sources into agreements, conflicts (usually explained by a moderating variable — the review's best contribution), and gaps. Searching the negation and stopping at saturation are what separate a review from confirmation shopping with citations.

## When to Use

- Grounding a decision, design, or paper in what's already known.
- Someone's about to spend a quarter building what a search might reveal is solved (or disproven).

**When NOT to use:**
- Adopt-a-tool decisions — `technology-evaluation`.

## Prerequisites

- The question the review serves, framed answerably (`research-question-framing` first if it's still "learn about X") — the question is every source's relevance filter.

## The Workflow

1. **Define scope as inclusion/exclusion rules before searching:** time window with a reason ("post-2017, transformer-era only"), domains in/out, evidence types accepted, and the stop-question: "what would make this review DONE?" Unscoped reviews don't finish; they lapse.

2. **Search in expanding rings, logging queries as you go:** seed terms → each field's synonyms (the vocabulary problem is real: "A/B testing" is "online controlled experiments" in academia) → the two power moves: **citation chasing** backward (whom do the good papers cite?) and forward ("cited by" finds the successors and the refutations), and venue browsing. The query log makes the search auditable and resumable. Search the negation explicitly ("X limitations", "X replication failure") — if you found no disagreement, you haven't looked.

3. **Triage in two passes, ruthlessly:** pass one — title+abstract against inclusion rules, minutes each (expect ~80% no); pass two — survivors get the structured read: skim method + results + limitations BEFORE any deep read (conclusion-only reading is how misreadings propagate — the "X improves Y 40%" that was n=12 undergrads). Log rejects' reasons in one line.

4. **Grade every source on two axes separately — rigor and relevance:** rigor tiers (meta-analyses > RCTs > observational > case reports > expert opinion > vendor content; practitioner: methodology-published benchmarks > experience reports > talks > posts) and relevance to YOUR question. Check the incentive layer: who funded it, what's being sold (`source-credibility-check` per source). Both high-rigor-low-relevance and low-rigor-high-relevance go in — labeled.

5. **Build the synthesis matrix — the step that converts reading into review:** rows = sources; columns = the aspects your question decomposes into (method, context, finding, effect size, limitations, grade). The matrix mechanically surfaces what prose hides: AGREEMENTS (consensus, with strength), CONFLICTS (the interesting part — usually explained by a moderating variable: different populations, scales, definitions), and GAPS.

6. **Write the synthesis by claims, not by sources:** organized around answers ("evidence strongly supports X in context A [5 sources, 2 RCTs]; evidence conflicts on B, apparently moderated by scale; no direct evidence on C"), each claim at the strength the evidence bears — not the annotated bibliography ("Smith says... Jones says...") that summarizes effort instead of knowledge.

7. **Stop at saturation, and timebox against it:** when new sources keep citing what you've read and stop adding claims to the matrix, the landscape is mapped — more reading is completionism. Ship with the freshness date + query log; reviews rot, and a fast-moving field gets a re-check trigger.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "I searched 'why X is better' and found strong support" | You harvested the agreement you queried for. The negation search is mandatory — every live topic has disagreement, and not finding it means not looking. |
| "Abstracts and conclusions cover the papers efficiently" | The abstract oversells; the limitations section confesses. Method-first skimming is where the n=12-undergrads discovery happens BEFORE the claim enters your matrix. |
| "Twenty sources summarized — the review is done" | Twenty summaries is an annotated bibliography: reading-proof, not knowledge. The matrix and claims-structure are where synthesis happens; without them you've alphabetized your effort. |
| "The sources conflict — the literature is just messy" | Conflicts usually resolve under a moderating variable (population, scale, definition) — and finding it is the review's single best contribution. 'Messy' is where the analysis stops one step early. |
| "One more week of reading will make it comprehensive" | Saturation already announced the end: new sources citing old ones, matrix rows adding nothing. Past that point, reading is procrastination with a bibliography. |
| "It's heavily cited, so the finding is solid" | Citation counts include refutations and reflexive cites of famously-failed-to-replicate work. Check whether the citations are supportive before the count means anything. |

## Red Flags

- No query log; the search unreproducible.
- Zero disagreeing sources in the final set.
- Claims quoted from abstracts with methods unread.
- The deliverable organized source-by-source.
- Vendor-funded findings ungraded for incentive.
- A review with no stated freshness date or stop rule, still "in progress" at month three.

## Verification

- [ ] Scope rules + stop-question written before searching — dated doc.
- [ ] Query log complete, including negation searches — attached.
- [ ] Two-pass triage counts recorded (screened → read) with reject reasons.
- [ ] Every included source graded on both axes; incentives noted — matrix column.
- [ ] Synthesis matrix built; conflicts resolved-or-named with candidate moderators.
- [ ] Deliverable organized by claims with evidence weights; freshness date + re-check trigger stated.

## Example

Question (decision-serving): "does trunk-based development measurably outperform long-lived feature branches for teams our size (~30 eng)?" — a quarter's process change hung on it. Scope: 2014+, empirical evidence only, team-scale contexts. Search rings: academic (MSR/ICSE) + practitioner (DORA, engineering blogs with data); the synonym ring mattered immediately ("continuous integration" vs "trunk-based" across communities). 61 sources triaged → 14 survived. Matrix columns: context/team size, metrics, finding, rigor. What the matrix surfaced: strong consensus that integration *frequency* correlates with delivery performance (DORA, multi-year, high rigor); conflict on whether branch *lifetime* or review *latency* is the active ingredient — the moderating-variable hunt showed the two negative studies had review latencies >2 days. Synthesis claim shipped: "evidence supports the switch IF review latency is fixed first — the branching change alone is unsupported." The team fixed review SLAs first, then switched; both moves' metrics landed as the review predicted. Total cost: four days, timeboxed.

## Related skills

- `research-question-framing` — sharpening the question before the search.
- `source-credibility-check` — the per-source audit inside step 4.
- `note-synthesis` — the note-taking layer feeding the matrix.
- `technology-evaluation` — the adopt-a-tool sibling.
