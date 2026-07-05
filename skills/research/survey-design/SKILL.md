---
name: survey-design
description: >
  Use when designing a survey or questionnaire — question wording that
  doesn't lead, scale choice, ordering effects, sampling honesty, pilot
  testing. Triggers: "design a survey", "write these survey questions",
  "NPS alternatives", "questionnaire for", "is this question biased",
  "survey response rate". NOT for qualitative depth — interviews find
  the what-you-didn't-know-to-ask; surveys quantify what interviews
  surfaced.
---

# Survey Design

## Overview

A shipped survey can't be patched — ambiguity found in 600 responses instead of 6 is permanent. Work backward from the analysis plan, write questions that don't smuggle answers, confront the sample honestly, and pilot with think-alouds: the instrument's validity is decided entirely before fielding.

## When to Use

- Building a survey whose answers feed a real decision.
- Auditing a draft questionnaire before it ships to 5,000 people.

**When NOT to use:**
- Discovering unknown unknowns — interviews (`user-interview-synthesis`); sequence surveys after them.

## Prerequisites

- The decision the results feed, and the analysis sketched IN ADVANCE ("compare segment A vs B on Q3") — questions without a planned analysis are curiosity taxes on respondents.
- Who the population is, and what sample you can actually reach.

## The Workflow

1. **Work backward from the analysis plan:** per intended finding, derive the exact question + segmentation fields — and nothing else. Every question costs response rate; the 40-question "while we're at it" survey buys worse data on all 40. Target: 5–12 questions.

2. **Write questions that don't smuggle answers:** one construct per question (the double-barrel "was it fast and reliable?" is unanswerable for fast-but-flaky — split it); neutral framing; no jargon or undefined windows ("recently" → "in the last 30 days"); recall bounded to rememberable spans (last week, not "on average" — people answer average-questions with self-image); behavior > intention ("which did you use last time" beats "which would you choose" — stated intent is weather).

3. **Choose response formats by the analysis, standardized:** 5-point fully-labeled Likert for attitudes; explicit units and non-overlapping ranges for numbers; "don't know / N/A" offered where genuinely possible (forcing manufactures data); option order randomized where not semantic; scale DIRECTION consistent throughout (mid-survey flips harvest attention errors).

4. **Order for the respondent's psychology:** easy on-topic openers (abandonment decides at question one); sensitive items (money, criticism, demographics) near the END; general-before-specific within topics (ask "checkout speed?" first and the subsequent "overall satisfaction?" is now a checkout-speed answer); demographics last, each justified by the segmentation plan.

5. **Confront sampling honestly — the step that governs what results MEAN:** who receives it, who responds, how both differ from the population; nonresponse bias worst when responding correlates with the topic (the angry and the delighted answer; the middle churns silently); incentives broaden AND bend. State the frame in the writeup ("611 of 4,000 invited; respondents skew tenured — treat prevalence as upper bound"). A biased sample honestly described is evidence; the same sample described as "our users" is fiction with a percentage sign.

6. **Pilot with 5–10 real respondents, thinking aloud:** watch them interpret each question (the construct you meant vs the one they answered — mismatches are silent in the data forever); time it honestly; dry-run the analysis on pilot data (can you compute the planned finding?). The pilot catches what author re-reading can't, because authors read their intent, not their words. A survey shipped unpiloted is a deploy without staging.

7. **Report with the machinery visible:** response rate and sample description, exact question wording IN the report (results divorced from wording get misquoted into stronger claims), don't-know rates per question (high DK = broken question), confidence intervals on headline percentages (n=611's ±4% changes how 52/48 reads). Segment cuts follow the pre-registered plan — post-hoc segment fishing has `ab-test-analysis` step 6's multiplicity arithmetic and the same rule: fished findings are next-instrument hypotheses.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "While we're fielding, let's add questions for the other teams" | Each added minute costs completions, and the 40-question omnibus buys worse data on everything. The analysis-plan filter is the instrument's spine. |
| "'How much easier is the new design?' — we need the enthusiasm" | You'll receive the enthusiasm you requested, and the decision built on it fails in production. Neutrality isn't politeness; it's measurement validity. |
| "'Would you pay $20/month?' tells us the price point" | The yes costs nothing and predicts nothing. Behavior questions or real WTP methods (Van Westendorp, conjoint); intent theater prices products for imaginary customers. |
| "300 responses from the feedback widget = what users want" | It's what widget-clickers want — the angry and the delighted, minus the silent middle. The frame sentence is the difference between evidence and marketing. |
| "No time to pilot — the questions read clearly to all of us" | Authors read intent, not words. The example's 'initial setup' meant billing to two pilots — caught at n=7 instead of poisoning n=640 permanently. |
| "Forcing an answer on every question gives complete data" | The respondent without an opinion picks SOMETHING, and you can't tell which rows are manufactured. Don't-know options keep the real data distinguishable. |

## Red Flags

- Questions with no line in the analysis plan.
- Double-barrels; endpoints-only labeled scales; overlapping ranges.
- Sensitive items in the first third.
- No pilot; no timing data.
- Results reported without wording, frame, or intervals.
- Segment findings shipped from cuts nobody pre-registered.

## Verification

- [ ] Analysis plan predates the instrument; every question maps to a planned finding — doc dated.
- [ ] Question audit passed: no double-barrels, neutral stems, bounded recall, behavior-over-intent where possible — reviewer noted.
- [ ] Formats standardized (labeled points, DK options, consistent direction) — instrument attached.
- [ ] Pilot done with think-alouds; wording fixes and analysis dry-run recorded.
- [ ] Sample frame stated in the report; response rate + skews described.
- [ ] Report includes exact wording, DK rates, CIs; segment cuts match the pre-registration.

## Example

Decision: prioritize onboarding fixes — quantify the interview finding (`user-interview-synthesis`'s delegation-permissions wall) across the base. Analysis plan first: prevalence of setup-delegation among new accounts, segmented by company size. Instrument: 8 questions. Draft audit caught: a double-barrel ("was setup quick and clear?") split; "how often do you typically delegate?" (self-image bait) replaced with "for THIS account, who performed the initial setup?" (behavioral, bounded); satisfaction moved before the specific setup questions; DK added. Pilot (n=7, think-aloud): "initial setup" was ambiguous — two pilots meant *billing*; reworded to name the three concrete steps. Fielded to 4,000 new accounts, 640 responses (16%; frame stated: skews toward activated accounts). Headline: 44% [±4] of ≥50-employee accounts had setup performed by someone other than the eventual daily user — the interview finding, now sized; the onboarding fix got its `prioritization-frameworks` Reach number from this cell.

## Related skills

- `user-interview-synthesis` — the qualitative pass that writes this instrument's questions.
- `ab-test-analysis` — shared statistics discipline (multiplicity, pre-registration).
- `metric-definition` — the precision habits for what gets counted.
- `experiment-design` — when you need causation, not measurement.
