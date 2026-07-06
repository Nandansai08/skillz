---
name: interview-prep-technical
description: >
  Use when preparing for technical interviews — gap analysis against the
  loop's actual formats, spaced practice scheduling, STAR story bank for
  behavioral rounds, mock calibration. Triggers: "prepare for my
  interview", "interview prep plan", "system design interview coming up",
  "behavioral questions prep", "I have an onsite in 3 weeks". NOT for
  the resume that got the interview (see resume-tailoring) or the offer
  after (see salary-negotiation).
---

# Interview Prep (Technical)

## Overview

Prep allocated by gap map beats prep allocated by comfort: the timed cold diagnostic per round type is humbling in a useful direction, and the budget goes to the round that decides the level, not the one that's fun. Patterns over problem-count, a story bank over improvisation, and mocks early enough to redirect — interviewing is a repeated game, and the notes compound.

## When to Use

- An interview loop is scheduled (or targeted) and prep time needs structuring.
- Prep is happening but shapeless — 200 random problems, no story bank, no mocks.

**When NOT to use:**
- The resume — `resume-tailoring`. The offer — `salary-negotiation`.

## Prerequisites

- The loop's actual format, researched: rounds, types, the company's known style — prep targeted at the wrong format is cardio for the wrong sport.

## The Workflow

1. **Gap-analyze against the loop, honestly:** per round type, a timed cold attempt (one medium problem, one 35-minute design prompt, one behavioral answer recorded) — the results ARE the gap map, usually humbling in a useful direction. Prep budget allocates to gaps weighted by round importance — not to the category that's most fun.

2. **Coding rounds: patterns over problem-count, under interview conditions:** practice by pattern family (`two-pointer-sliding-window`, `graph-traversal-patterns`, `dynamic-programming-derivation`, `heap-and-priority-patterns`, `binary-search-variants` ARE the curriculum) — 60–100 problems covering patterns beats 300 random; every problem timed, talking aloud, bare editor. The post-problem ritual is where learning lives: could I state the pattern? What trigger did I miss? Redo fails from scratch 3+ days later (spaced repetition on the misses, not the wins).

3. **System design: a repeatable frame plus real-system depth:** drill one skeleton until automatic — requirements+scale first (the numbers change everything), API + data model, high-level shape, deep-dive where steered, trade-offs stated proactively — fed by real engineering (`capacity-planning` estimation habits, `api-design-rest`, postmortem-reading for how systems actually fail). The differentiator interviewers report: candidates who state trade-offs unprompted versus those who recite architectures.

4. **Build the STAR story bank — 8–12 stories, indexed to question families:** Situation/Task in 2 sentences, Action as YOUR decisions (the "we" that hides you is the genre's classic failure — interviewers grade your role), Result with numbers, plus a reflection line (senior signals live there). Index against the families (conflict, failure, leadership, ambiguity, deadline) — good stories serve 3+ families with different emphasis. Source from the `promotion-packet`-style evidence review — the material exists, unmined.

5. **Mock under real conditions, early enough to act on findings:** 2–3 mocks per weak round, harshest available reviewer, recorded where possible — the recording finds the verbal tics, the silence-instead-of-narrating, the solution rushed before requirements. First mock at ~30% through prep, not the final week: mocks are the diagnostic that redirects prep, wasted as a graduation ceremony.

6. **Schedule with spacing and a taper:** consistent daily blocks beat weekend binges (`deep-work-scheduling` supplies them); interleave categories within weeks; the final week TAPERS — pattern-list review, story bank, logistics, sleep. Cramming new material into the last 48 hours buys anxiety, not capability.

7. **Run the loop as data collection, then retro:** during — requirements questions before solving, thinking narrated, hints taken gracefully (interviewers grade collaboration; fighting a hint fails candidates who'd have passed); after each round — 10 minutes of notes while fresh; after the loop — the retro against the gap map, feeding the next loop's step 1. The notes compound.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "300 problems solved — I'm ready" | Volume without pattern extraction and timed conditions grows the recognition skill slowly. The 60-pattern-organized problems with the post-problem ritual is what the interview actually tests. |
| "I'll focus on coding — it's where I'm strongest" | Comfort-zone allocation: the failing round is usually the unpracticed one, and system design decides the level. The gap map allocates; enjoyment doesn't. |
| "Behavioral is just telling them what happened" | Unprepped answers run four minutes of resultless 'we.' The bank is two evenings; composing under adrenaline is a choice with a known failure rate. |
| "I'll mock right before the onsite as a final check" | The final-week mock finds problems too late to fix. At 30% through, it's the diagnostic that redirects the remaining 70%; at 95%, it's a preview of the bill. |
| "Solving silently is fine — the code speaks" | The interviewer graded a black box and passed on it. Narration is a trained skill; the recordings train it, and untrained it fails candidates whose code was right. |
| "Last 48 hours: cram the weak patterns" | Day −2 material doesn't consolidate; depleted arrival does. The taper is counterintuitive and correct — capability was built in the weeks; the final days protect it. |

## Red Flags

- Prep hours logged with no gap map anywhere.
- Problem practice with an IDE's autocomplete and no timer.
- Zero recorded/mocked behavioral answers.
- System design "prep" that's watching videos without running the skeleton.
- First mock scheduled inside the final week.
- No notes after real rounds; each loop starting from scratch.

## Verification

- [ ] Cold diagnostics done per round type; gap map with budget allocation written.
- [ ] Coding practice logged by pattern family with the miss-redo list active.
- [ ] Design skeleton run ≥8 times against varied prompts — trade-off statements practiced.
- [ ] Story bank: 8–12 STAR stories, indexed, results quantified — doc exists.
- [ ] ≥2 mocks per weak round completed by 50% of prep; findings acted on — noted.
- [ ] Taper week planned; post-round note template ready.

## Example

Candidate: senior backend engineer, 4 weeks to a loop (2 coding, 1 system design, 1 behavioral, 1 domain). Cold diagnostics: coding medium solved but 8 minutes over, narration absent; design prompt wandered requirements-less for 15 minutes; behavioral answer 4 minutes of "we." Gap map allocated: 40% system design (the level-decider), 30% coding-under-conditions, 20% behavioral bank, 10% domain. Weeks 1–3: daily 90-minute blocks interleaved — coding by pattern with the redo-misses ritual (top miss-log entry: sliding-window monotonicity checks — drilled); design skeleton run 11 times, trade-off-stating practiced explicitly; story bank built (9 stories; the "failure" story sourced from a postmortem the candidate had written — pre-quantified). Mocks at days 8 and 19: mock 1's recording exposed the silence habit (fixed by narrating even dead ends); mock 2's design feedback ("you never asked about read/write ratio") patched the skeleton. Final week: taper. Result: offer; the round notes recorded that the domain round asked exactly one deep question the 10% allocation had covered — and the retro file made the NEXT loop's prep, a year later, a two-week affair.

## Related skills

- `resume-tailoring` — upstream; the claims this prep must defend.
- `salary-negotiation` — downstream; the offer this prep produces.
- `two-pointer-sliding-window` / `graph-traversal-patterns` / `dynamic-programming-derivation` / `binary-search-variants` / `heap-and-priority-patterns` — the coding curriculum.
- `deep-work-scheduling` — the practice blocks' construction.
