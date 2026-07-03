---
name: survey-design
description: >
  Use when designing a survey or questionnaire — question wording that
  doesn't lead, scale choice, ordering effects, sampling honesty, pilot
  testing. Triggers: "design a survey", "write these survey questions",
  "NPS alternatives", "questionnaire for", "is this question biased",
  "survey response rate".
---

# Survey Design

## When to use this skill
- Building a survey whose answers will feed a real decision (user research, employee feedback, market sizing).
- Auditing someone's draft questionnaire before it ships to 5,000 people (unlike code, a shipped survey can't be patched).
- NOT for qualitative depth — interviews (`user-interview-synthesis`) find the *what-you-didn't-know-to-ask*; surveys quantify what interviews surfaced. Sequence them that way.

## Prerequisites
- The decision the results feed, and the analysis sketched IN ADVANCE ("we'll compare segment A vs B on question 3") — questions without a planned analysis are curiosity taxes on respondents.
- Who the population is, and what sample of it you can actually reach (the honesty about this gap shapes every claim you'll be allowed to make).

## Workflow

1. **Work backward from the analysis plan:** for each intended finding ("X% of segment S report pain P"), derive the exact question + the segmentation fields needed — and nothing else. Every question costs response rate (completion drops measurably per added minute) and analysis time; the 40-question "while we're at it" survey buys worse data on all 40. Target: shortest instrument that feeds the plan, usually 5–12 questions.

2. **Write questions that don't smuggle answers:** one construct per question (the double-barrel "was the product fast and reliable?" is unanswerable for fast-but-flaky users — split it); neutral framing (not "how much do you love..." — and the subtler version: mentioning ONE option's benefit in the stem); no unfamiliar jargon or undefined time windows ("recently" → "in the last 30 days"); recall questions bounded to plausibly-rememberable windows (behavior last week, not "on average" — people answer "average" questions with self-image, not arithmetic); and behavior > intention wherever possible ("which did you use last time" beats "which would you choose" — stated intent is weather, per the interview skill's same rule).

3. **Choose response formats by the analysis, standardized:** 5-point labeled Likert for attitudes (7 adds noise, not resolution; label EVERY point, not just endpoints); explicit units and non-overlapping ranges for numbers (the "1–5 / 5–10" boundary double-count); "not applicable / don't know" offered where genuinely possible (forcing an answer manufactures data — the respondent without an opinion WILL pick something); randomize option order where order isn't semantic (primacy bias is real); keep scale *direction* consistent through the instrument (flipping agree-left to agree-right mid-survey harvests attention errors, and "attention-check by reversal" needs doing deliberately, not accidentally).

4. **Order for the respondent's psychology:** easy, engaging, on-topic openers (abandon-decisions happen in question one); sensitive items (money, criticism, demographics) near the END — early sensitive questions cost completions AND bias what follows; general-before-specific within topics (the specific question contaminates the general one asked after it: prime someone with "how was checkout speed?" and their subsequent "overall satisfaction?" is now a checkout-speed answer); demographics last, minimal, each justified by the segmentation plan.

5. **Confront sampling honestly — the step that governs what the results MEAN:** who receives it, who responds, and how both differ from the population (the survey of your newsletter subscribers describes your newsletter subscribers); nonresponse bias worst when responding correlates with the topic (the angry and the delighted answer; the middle churns silently); incentives broaden samples but bend them; state the frame in the writeup ("of 4,000 invited active users, 611 responded; respondents skew tenured — treat prevalence estimates as upper bounds for engaged users"). A biased sample honestly described is usable evidence; the same sample described as "our users" is fiction with a percentage sign.

6. **Pilot with 5–10 real respondents, thinking aloud:** watch them interpret each question (the construct you meant vs the one they answered — mismatches are silent in the data forever after), time it honestly, check the analysis dry-run on pilot data (can you actually compute the planned finding? the un-analyzable question is cheaper to fix now). The pilot catches the ambiguity that no amount of author re-reading can, because authors read their intent, not their words. A survey shipped unpiloted is a deploy without staging.

7. **Report with the machinery visible:** response rate and sample description (step 5's honesty), exact question wording IN the report (results divorced from wording get misquoted into stronger claims), "don't know" rates per question (high DK = the question or the construct was broken), and confidence intervals on the headline percentages (n=611's ±4% changes how a 52/48 "majority" reads). Segment cuts follow the pre-registered plan — post-hoc segment fishing has the same multiplicity arithmetic as `ab-test-analysis` step 6, and the same rule: fished findings are hypotheses for the NEXT instrument.

## Common pitfalls
- Leading-question harvest: "how much easier is the new design?" produces the enthusiasm it requested; the decision made on it fails in production. Neutrality (step 2) isn't politeness, it's measurement validity.
- The double-barrel: one question, two constructs, uninterpretable answers — and it's the single most common defect in amateur instruments.
- Intent theater: "would you pay $20/month?" — the yes costs nothing and predicts nothing; last-purchase behavior questions or actual willingness-to-pay methods (Van Westendorp, conjoint) exist for this.
- Sample laundering: 300 volunteers from a feedback widget reported as "users want" — the frame sentence (step 5) is the difference between evidence and marketing.
- Survey-as-interview-substitute: fielding a questionnaire to discover unknown unknowns — closed questions can only count what you already knew to ask; the open-text box at the end is a weak apology for skipped interviews.
- Skipping the pilot to hit a deadline — then discovering question 4's ambiguity in 600 responses instead of 6, with no patch possible.

## Example
Decision: prioritize onboarding fixes — quantify the interview finding (`user-interview-synthesis`'s delegation-permissions wall) across the base. Analysis plan first: prevalence of setup-delegation among new accounts, segmented by company size; plus time-to-value self-report. Instrument: 8 questions. Draft audit caught: a double-barrel ("was setup quick and clear?") split in two; "how often do you typically delegate technical setup?" (self-image bait) replaced with "for THIS account, who performed the initial setup?" (behavioral, bounded); satisfaction item moved before the specific setup questions (contamination order); "don't know" added to the permissions question. Pilot (n=7, think-aloud): "initial setup" was ambiguous — two pilots meant *billing* setup; reworded to name the three concrete steps. Fielded to 4,000 new accounts, 640 responses (16%; frame stated: skews toward accounts that activated ≥once). Headline: 44% [±4] of ≥50-employee accounts had setup performed by someone other than the eventual daily user — the interview finding, now sized and segmented; the onboarding fix got its `prioritization-frameworks` Reach number from this cell, and the pre-registered size-segmentation is what made the enterprise-onboarding investment case in one slide.

## Related skills
- `user-interview-synthesis` — the qualitative pass that writes this instrument's questions.
- `ab-test-analysis` — shared statistics discipline (multiplicity, pre-registration).
- `metric-definition` — the precision habits for what gets counted.
- `experiment-design` — when you need causation, not measurement.
