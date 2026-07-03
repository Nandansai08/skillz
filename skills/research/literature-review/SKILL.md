---
name: literature-review
description: >
  Use when surveying what's known on a topic — systematic search strategy,
  source quality tiers, synthesis matrices, knowing when to stop reading.
  Triggers: "literature review on", "what's the state of the art",
  "survey the research on", "what's known about", "before we reinvent this,
  what exists".
---

# Literature Review

## When to use this skill
- Grounding a decision, design, or paper in what's already known — academic or practitioner topics both.
- Someone's about to spend a quarter building what a search might reveal is solved (or disproven).
- NOT for evaluating a specific tool/library for adoption — that's `technology-evaluation`; this skill maps a knowledge landscape.

## Prerequisites
- The question the review serves, framed answerably (`research-question-framing` first if it's still "learn about X") — the question is the relevance filter every source gets tested against.

## Workflow

1. **Define scope as inclusion/exclusion rules before searching:** time window (and why — "post-2017, transformer-era only" is a reasoned cut; "recent stuff" isn't), domains in/out, evidence types accepted (RCTs? benchmarks? case studies? practitioner reports?), and the stop-question: "what would make this review DONE?" Unscoped reviews don't finish; they lapse.

2. **Search in expanding rings, logging queries as you go:** seed terms → each field's synonyms (the vocabulary problem is real: "A/B testing" is "online controlled experiments" in academia, "split testing" in marketing) → then the two power moves: **citation chasing** backward (who do the good papers cite?) and forward (who cites THEM — Semantic Scholar/Google Scholar's "cited by" finds the successors and the refutations), and venue browsing (the 2–3 journals/conferences/blogs where this topic lives). The query log makes the search auditable and resumable — and shows a reviewer you didn't just read page one of one search.

3. **Triage in two passes, ruthlessly:** pass one — title+abstract against the inclusion rules, minutes each, sorting into read/maybe/no (expect ~80% no); pass two — the survivors get the structured read: skim method + results + limitations before any deep read (conclusion-only reading is how misreadings propagate). Log the rejects' reasons in one line — "excluded: pre-2017" prevents re-triaging the same paper in week three.

4. **Grade every source on the two axes separately — rigor and relevance:** rigor tiers (meta-analyses/systematic reviews > RCTs/controlled experiments > observational studies > case reports > expert opinion > vendor content — with practitioner topics adding: benchmarks-with-published-methodology > experience reports > conference talks > blog posts), and relevance to YOUR question (a rigorous study of adjacent conditions can matter less than a modest study of your exact ones). Check the incentive layer: who funded it, what's being sold, whether the "independent benchmark" has a logo on it. High-rigor-low-relevance and low-rigor-high-relevance both go in — labeled.

5. **Build the synthesis matrix — the step that converts reading into review:** rows = sources; columns = the aspects your question decomposes into (method, population/context, key finding, effect size, limitations, quality grade). The matrix mechanically surfaces what prose reading hides: where sources AGREE (consensus, with strength), where they CONFLICT (the interesting part — usually explained by a moderating variable: different populations, scales, definitions; finding that variable is the review's best contribution), and where nobody has looked (the gap).

6. **Write the synthesis by claims, not by sources:** organized around answers to the question ("evidence strongly supports X in context A [5 sources, incl. 2 RCTs]; evidence conflicts on B, apparently moderated by scale [3 vs 2, see matrix]; no direct evidence on C — nearest adjacent finding is..."), each claim carrying its evidence weight honestly — not the annotated bibliography ("Smith says... Jones says...") that summarizes reading effort instead of knowledge. State conclusions at the strength the evidence bears, including "unknown."

7. **Stop at saturation, and timebox against it:** when new sources keep citing what you've already read and stop adding claims to the matrix, the landscape is mapped — more reading is completionism. Set the timebox upfront (a decision-serving review: days; a publication-grade one: longer), and ship with the freshness date + the query log, because reviews rot (set a re-check trigger if the field moves fast — the review that quietly aged two years misleads with citations).

## Common pitfalls
- Confirmation-seeded search: querying "why X is better" and reviewing the agreement you harvested. Search the negation explicitly ("X limitations", "X vs", "X replication failure") — if you found no disagreement, you haven't looked, since every live topic has some.
- Abstract-and-conclusion reading: the abstract oversells, the limitations section confesses — the paper whose "X improves Y 40%" was n=12 undergrads in a lab. The structured skim (step 3) reads methods first.
- Authority laundering: "studies show" citing a blog citing a press release citing a preprint that says something narrower. Chase claims to their primary source; secondary citations mutate (`source-credibility-check` is the per-source deep dive).
- The annotated-bibliography deliverable: forty faithful summaries, zero synthesis — the matrix (step 5) and claims-structure (step 6) are the difference between reading-proof and a review.
- Vocabulary-blindness: searching only your field's term for a concept three fields have studied under other names (step 2's synonym ring) — the "gap" that's actually a translation failure.
- No stop rule: month three, source ninety, the decision the review was serving got made without it in week four. Timebox + saturation (step 7).

## Example
Question (decision-serving): "does trunk-based development measurably outperform long-lived feature branches for teams our size (~30 eng)?" — a quarter's process change hung on it. Scope: 2014+, empirical evidence only (opinion pieces excluded), team-scale contexts. Search rings: academic (MSR/ICSE venues) + practitioner (DORA reports, engineering blogs with data); the synonym ring mattered immediately ("continuous integration" in academia ≈ practice-adjacent; "trunk-based" mostly practitioner vocabulary). 61 sources triaged → 14 survived. Matrix columns: context/team size, metrics used, finding, rigor tier. What the matrix surfaced: strong consensus that integration *frequency* correlates with delivery performance (DORA, multi-year, high rigor); conflict on whether branch *lifetime* or review *latency* is the active ingredient — the moderating-variable hunt showed the two negative studies had review latencies >2 days, suggesting the bottleneck was review, not branching. Synthesis claim shipped: "evidence supports the switch IF review latency is fixed first — the branching change alone is unsupported." The team fixed review SLAs first, then switched; both moves' metrics landed as the review predicted. Total review cost: four days, timeboxed.

## Related skills
- `research-question-framing` — sharpening the question before the search.
- `source-credibility-check` — the per-source audit inside step 4.
- `note-synthesis` — the note-taking layer feeding the matrix.
- `technology-evaluation` — the adopt-a-tool sibling.
