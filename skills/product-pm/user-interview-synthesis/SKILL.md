---
name: user-interview-synthesis
description: >
  Use when turning user-interview notes into findings — coding transcripts
  into themes, separating observation from interpretation, resisting
  cherry-picking, producing decision-ready insights. Triggers: "synthesize
  these interviews", "what did we learn from the research", "affinity
  mapping", "turn interview notes into insights", "research readout".
---

# User Interview Synthesis

## When to use this skill
- A batch of interviews (usually 5–12) is done and must become findings people can act on.
- Auditing a research readout that smells like confirmation ("users love our roadmap!").
- NOT for designing the interviews themselves — question construction and sampling live in `survey-design`'s neighborhood and standard research practice; garbage in survives any synthesis.

## Prerequisites
- Raw material: transcripts or detailed notes (recordings >> memory — synthesis from recollection synthesizes your biases); the research question the interviews were run to answer.

## Workflow

1. **Separate the two layers before anything: what was SAID vs what you think it MEANS.** Pass one through each transcript extracts observations — verbatim quotes and described behaviors, tagged to participant ("P3: 'I export to Excel and fix the fees by hand, every Friday' — behavior: weekly manual workaround"). No interpretation enters this layer. The discipline sounds pedantic and is the entire method: every synthesis failure downstream is interpretation smuggled into the evidence layer.

2. **Weight behavior over opinion, systematically:** what users DO (current workarounds, actual sequences, things they've built) is strong evidence; what they SAY they'd do ("I would definitely pay for that") is weather. Tag each observation as behavior / experienced-pain / preference / speculation — and let the speculative tier inform hypotheses only, never conclusions. The user who built a spreadsheet macro has voted; the user who "loves the idea" has smiled.

3. **Code bottom-up into themes; let the structure emerge:** cluster observations by affinity (physical/digital cards both work), naming clusters *after* they form rather than sorting into pre-made buckets — pre-made buckets find only what you pre-made. Every cluster keeps its participant IDs attached: a "theme" that's three quotes from one vivid participant is an anecdote wearing a trenchcoat.

4. **Hunt your own confirmation bias as a step, not a mood:** actively list the observations that contradict the team's going hypothesis (there are always some) and give them their own section; check whether the vivid quote you keep repeating is representative or just quotable; note which participants you found "insightful" (usually the ones who agreed) and re-read the others. If the synthesis confirms everything the team already believed, either the product intuition is superb or the synthesis is a mirror — say which, with evidence.

5. **Write findings as evidence-graded claims:** each finding = claim + strength + evidence trail. "Manual fee reconciliation is a weekly time sink for ops-role users (STRONG: behavior observed in 6/8, incl. two self-built workarounds — P2's macro, P7's template)" vs "Users may want API access (WEAK: unprompted mention by 2/8, speculative tier)." The n/N notation with tier does honest work: 8 interviews can establish "this pain exists and runs deep" — it cannot establish "34% of the market"; findings must not borrow quantitative costume (`survey-design` is where quantification lives).

6. **Bridge each finding to a decision or a question — the So-What column:** finding → implication → recommended action or follow-up ("STRONG reconciliation pain → validates the PRD's problem statement → proceed; WEAK API interest → add a question to the next survey wave, don't build"). Findings without implications are trivia; the readout's job is feeding `prd-writing` step 1 and `prioritization-frameworks` step 2's Confidence tiers with honestly-graded evidence.

7. **Ship the readout in two layers and archive the trail:** a one-page decision layer (research question, 3–5 graded findings, contradicting-evidence section, so-whats) for the people deciding; the appendix (themes, quotes, per-participant summaries) for the people who'll ask "based on what?" Archive transcripts + coding searchably — next quarter's question will partially overlap, and re-listening beats re-interviewing.

## Common pitfalls
- Synthesis-by-highlight-reel: the readout is eight great quotes agreeing with the roadmap. Quotes are illustrations of findings, never the findings; the contradiction section (step 4) is the tell of honest work — its absence is the tell of the other kind.
- Counting heads on opinions ("5 of 8 liked concept B") as if interviews were surveys — small-n preference counts are noise with confidence; behaviors and pains generalize from small n, votes don't.
- The vivid-participant distortion: P3 was articulate and quotable, and somehow every theme stars P3. Participant IDs on clusters (step 3) make this visible mechanically.
- Leading synthesis downstream of leading questions ("would you use X?" — everyone says yes politely) — flag solicited enthusiasm in the speculation tier; better, fix it upstream in interview design.
- Findings that outrun the sample: "users want" from n=8 of one segment, recruited from your happiest customers. State the sample's shape in the readout header; let readers see the lens.
- Synthesis six weeks later from cold notes — coding rots fast; the observation layer (step 1) done within days of each interview keeps the evidence evidence.

## Example
Research question: "why do trials stall before the aha moment?" — 9 interviews, stalled-trial users. Observation pass: 214 tagged items. Affinity coding produced five clusters; the headline theme wasn't the hypothesized onboarding confusion — 7/9 participants had *delegated setup to a colleague who lacked permissions* (behavior tier: observed in 4 screen-shares, described by 3 more), then never returned. The contradiction section did its job: the team's going theory (tutorial too long) had support from only 2/9, both speculation-tier ("I guess I'd want it shorter?" — solicited). Findings shipped graded: STRONG delegation-permissions wall (7/9, behavioral) → implication: the invite-flow permission default is the funnel's real gate → PRD problem statement rewritten around it; WEAK tutorial length → survey question added to the next wave, tutorial rebuild descoped. The permissions fix shipped in three weeks and moved trial-conversion +18% (`ab-test-analysis`-confirmed) — the synthesis's value being precisely the theme nobody went looking for, surfaced because the coding was bottom-up and the mirror-check ran.

## Related skills
- `survey-design` — quantifying what interviews surfaced.
- `prd-writing` — the evidence pipeline's destination.
- `prioritization-frameworks` — Confidence tiers fed by graded findings.
- `note-synthesis` — the same claims-from-sources discipline, solo-scale.
