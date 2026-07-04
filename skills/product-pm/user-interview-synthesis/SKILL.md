---
name: user-interview-synthesis
description: >
  Use when turning user-interview notes into findings — coding
  transcripts into themes, separating observation from interpretation,
  resisting cherry-picking, producing decision-ready insights. Triggers:
  "synthesize these interviews", "what did we learn from the research",
  "affinity mapping", "turn interview notes into insights", "research
  readout". NOT for designing the interviews themselves (question
  construction and sampling live upstream) — garbage in survives any
  synthesis.
---

# User Interview Synthesis

## Overview

Synthesis lives or dies on one discipline: what was SAID (observations, verbatim, tagged) stays separate from what you think it MEANS. Behavior outweighs opinion, themes emerge bottom-up, the contradiction section proves the work was honest — and small-n findings ship graded, never wearing quantitative costume.

## When to Use

- A batch of interviews (5–12) must become findings people can act on.
- Auditing a research readout that smells like confirmation.

**When NOT to use:**
- Designing the instrument — upstream; `survey-design` for the quantified sibling.

## Prerequisites

- Raw material: transcripts or detailed notes (recordings >> memory — synthesis from recollection synthesizes your biases); the research question the interviews were run to answer.

## The Workflow

1. **Separate the two layers before anything: what was SAID vs what you think it MEANS.** Pass one extracts observations — verbatim quotes and described behaviors, tagged to participant ("P3: 'I export to Excel and fix the fees by hand, every Friday' — behavior: weekly manual workaround"). No interpretation enters this layer. The discipline sounds pedantic and is the entire method: every downstream synthesis failure is interpretation smuggled into evidence.

2. **Weight behavior over opinion, systematically:** what users DO (workarounds, actual sequences, things they've built) is strong evidence; what they SAY they'd do ("I'd definitely pay for that") is weather. Tag each observation: behavior / experienced-pain / preference / speculation — the speculative tier informs hypotheses, never conclusions. The user who built a spreadsheet macro has voted; the user who "loves the idea" has smiled.

3. **Code bottom-up into themes; let the structure emerge:** cluster observations by affinity, naming clusters *after* they form — pre-made buckets find only what you pre-made. Every cluster keeps its participant IDs attached: a "theme" that's three quotes from one vivid participant is an anecdote wearing a trenchcoat.

4. **Hunt your own confirmation bias as a step, not a mood:** actively list observations that contradict the team's going hypothesis (there are always some) and give them their own section; check whether the quote you keep repeating is representative or just quotable. If the synthesis confirms everything the team already believed, either the intuition is superb or the synthesis is a mirror — say which, with evidence.

5. **Write findings as evidence-graded claims:** claim + strength + trail. "Manual fee reconciliation is a weekly time sink for ops users (STRONG: behavior observed in 6/8, incl. two self-built workarounds)" vs "Users may want API access (WEAK: unprompted mention by 2/8, speculative)." The n/N-with-tier notation does honest work: 8 interviews establish "this pain exists and runs deep" — never "34% of the market" (`survey-design` is where quantification lives).

6. **Bridge each finding to a decision or a question — the So-What column:** finding → implication → action or follow-up ("STRONG reconciliation pain → validates the PRD's problem statement → proceed; WEAK API interest → survey question, don't build"). Findings without implications are trivia; the readout feeds `prd-writing` step 1 and `prioritization-frameworks`' Confidence tiers.

7. **Ship the readout in two layers and archive the trail:** a one-page decision layer (question, 3–5 graded findings, contradiction section, so-whats) for deciders; the appendix (themes, quotes, per-participant summaries) for "based on what?" Archive transcripts + coding searchably — next quarter's question overlaps, and re-listening beats re-interviewing. State the sample's shape in the header — readers see the lens.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The quotes tell the story — lead with the best ones" | A highlight reel of agreeing quotes is confirmation with production values. Quotes illustrate findings; the contradiction section is the tell of honest work. |
| "5 of 8 preferred concept B — that's a majority" | Head-counting opinions at n=8 is noise wearing arithmetic. Behaviors and pains generalize from small n; votes don't (step 5's grading exists for this line). |
| "P3 explained it perfectly — their quotes carry the readout" | The vivid-participant distortion: articulate ≠ representative. Participant IDs on clusters make one-voice 'themes' mechanically visible. |
| "Everyone said yes when we asked if they'd use it" | Solicited enthusiasm to a would-you question is politeness data — speculation tier at best. What they've BUILT and DONE is the evidence layer. |
| "Synthesis can wait until the project needs it" | Coding rots in weeks: the observation layer done days after each interview keeps evidence evidence; the month-later version synthesizes your memory of your impressions. |
| "The findings confirm our roadmap — great news, ship the readout" | Perfect confirmation is the mirror-warning (step 4). Name it, hunt the disconfirming observations, and only then let the readout bless anything. |

## Red Flags

- A readout with zero contradicting evidence anywhere.
- Percentages computed on n<20 opinions.
- One participant ID dominating multiple themes.
- Findings ungraded; speculation and behavior interleaved as equals.
- No sample-shape statement (who was recruited, from where).
- Raw notes discarded after the readout shipped.

## Verification

- [ ] Observation layer exists, verbatim + participant-tagged, interpretation-free — sample shown.
- [ ] Every observation tier-tagged (behavior/pain/preference/speculation).
- [ ] Themes emerged bottom-up; each cluster's participant spread visible.
- [ ] Contradiction section present with real entries.
- [ ] Findings graded with n/N + tier; no market-share costume on small-n claims.
- [ ] So-what column complete; readout's sample-shape stated in the header — one-pager linked.

## Example

Research question: "why do trials stall before the aha moment?" — 9 interviews, stalled-trial users. Observation pass: 214 tagged items. Affinity coding produced five clusters; the headline theme wasn't the hypothesized onboarding confusion — 7/9 participants had *delegated setup to a colleague who lacked permissions* (behavior tier: observed in 4 screen-shares, described by 3 more), then never returned. The contradiction section did its job: the team's going theory (tutorial too long) had support from only 2/9, both speculation-tier. Findings shipped graded: STRONG delegation-permissions wall → the invite-flow permission default is the funnel's real gate → PRD problem statement rewritten around it; WEAK tutorial length → survey question added, rebuild descoped. The permissions fix shipped in three weeks and moved trial-conversion +18% (`ab-test-analysis`-confirmed) — the synthesis's value being precisely the theme nobody went looking for.

## Related skills

- `survey-design` — quantifying what interviews surfaced.
- `prd-writing` — the evidence pipeline's destination.
- `prioritization-frameworks` — Confidence tiers fed by graded findings.
- `note-synthesis` — the same claims-from-sources discipline, solo-scale.
