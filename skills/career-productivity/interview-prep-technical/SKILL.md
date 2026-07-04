---
name: interview-prep-technical
description: >
  Use when preparing for technical interviews — gap analysis against the
  loop's actual formats, spaced practice scheduling, STAR story bank for
  behavioral rounds, mock calibration. Triggers: "prepare for my interview",
  "interview prep plan", "system design interview coming up", "behavioral
  questions prep", "I have an onsite in 3 weeks".
---

# Interview Prep (Technical)

## When to use this skill
- An interview loop is scheduled (or targeted) and prep time needs structuring.
- Prep is happening but shapeless — 200 random LeetCode problems, no behavioral bank, no mocks.
- NOT for the resume that got the interview (`resume-tailoring`) or the offer after it (`salary-negotiation`).

## Prerequisites
- The loop's actual format, researched: rounds, types (coding/system design/behavioral/domain), and the company's known style (recruiter asks, candidate reports on Glassdoor/Blind, the company's own interview guides — prep targeted at the wrong format is cardio for the wrong sport).

## Workflow

1. **Gap-analyze against the loop, honestly:** per round type, self-assess current-vs-needed (a timed cold attempt per category is the honest assessor — one medium coding problem, one 35-minute system design prompt, one behavioral answer recorded; the results ARE the gap map, and they're usually humbling in a useful direction). Prep budget allocates to the gaps weighted by round importance — not to the category that's most fun (the strong coder grinding more coding while system design, the actual decider for their level, goes unpracticed).

2. **Coding rounds: patterns over problem-count, under interview conditions:** organize practice by pattern family (the `two-pointer-sliding-window`, `graph-traversal-patterns`, `dynamic-programming-derivation`, `heap-and-priority-patterns`, `binary-search-variants` skills ARE the curriculum) — 60–100 problems covering patterns beats 300 random; every problem timed, talking aloud, in a bare editor (no autocomplete, no run-until-green — the interview's conditions); the post-problem ritual is where learning lives: could I state the pattern? What was the trigger I missed? Redo fails from scratch 3+ days later (spaced repetition on the misses, not the wins).

3. **System design: a repeatable frame plus real-system depth:** drill one skeleton until automatic — requirements+scale first (the numbers change everything), API + data model, high-level shape, deep-dive where the interviewer steers, bottlenecks and trade-offs stated proactively — then feed it with real engineering (`capacity-planning`'s estimation habits, `api-design-rest`, `medallion-architecture-design`-style storage thinking, and postmortem-reading for how systems actually fail). The differentiator interviewers report: candidates who state trade-offs unprompted ("consistent hashing here costs us X; at this scale I'd accept it because Y") versus those who recite architectures.

4. **Build the STAR story bank — 8–12 stories, indexed to question families:** each story: Situation/Task compressed to 2 sentences, Action as YOUR decisions (the "we" that hides you is the genre's classic failure — interviewers grade your role, not the team's outcome), Result with numbers, plus a reflection line (what you'd do differently — senior signals live there). Index each story against the standard families (conflict, failure, leadership, ambiguity, deadline, disagreement-with-authority) — most good stories serve 3+ families with different emphasis; the bank means never composing under pressure. Source the stories from the `promotion-packet`-style evidence review — the material exists, unmined.

5. **Mock under real conditions, early enough to act on findings:** at least 2–3 mocks per weak round type, with the harshest available reviewer (peer engineer > friendly friend; paid platforms fill gaps), recorded where possible — the recording review finds the verbal tics, the silence-instead-of-thinking-aloud, the solution rushed before requirements (`ui-heuristic-review`-grade findings about yourself). First mock at ~30% through prep, not the final week: mocks are the diagnostic that redirects prep, wasted as a graduation ceremony.

6. **Schedule with spacing and a taper:** consistent daily blocks beat weekend binges (retention is spacing's product — `deep-work-scheduling` supplies the blocks); interleave categories within a week rather than serial monoliths (a month of coding then a "system design week" leaves coding rusty at the loop); the final week TAPERS — light review of the pattern list and story bank, logistics, sleep — cramming new material into the last 48 hours buys anxiety, not capability.

7. **Run the loop as data collection, then retro:** during — requirements questions before solving, thinking narrated, hints taken gracefully (interviewers grade collaboration; fighting a hint fails candidates who'd have passed); after each round — 10 minutes of notes while fresh (questions asked, where it wobbled); after the loop — the retro against the gap map (which prep paid, which round surprised) feeding the next loop's step 1, because interviewing is a repeated game and the notes compound (`blameless-postmortem` energy, career edition).

## Common pitfalls
- Random-problem grinding: 300 problems, no pattern extraction, no timed conditions — volume mistaken for prep while the recognition skill (the actual tested thing) grows slowly (step 2's ritual is the difference).
- Comfort-zone allocation: prepping the strong suit because progress feels good there — the gap map (step 1) exists because the failing round is usually the unpracticed one.
- Behavioral improvisation: "I'll just tell them what happened" — rambling, we-heavy, resultless answers to fully predictable questions. The bank (step 4) is two evenings of work; composing live under adrenaline is a choice.
- Mock avoidance: solo prep until the real loop becomes the first mock — at full price. The ego cost of an early harsh mock is the cheapest tuition available (step 5).
- Silent solving: perfect solution, zero narration — the interviewer graded a black box and passed on it. Thinking aloud is a trained skill, not a personality trait; the recordings train it.
- Last-week cramming: new patterns at day −2, arriving depleted — the taper (step 6) is counterintuitive and correct.

## Example
Candidate: senior backend engineer, 4 weeks to a loop (2 coding, 1 system design, 1 behavioral, 1 domain). Cold diagnostics: coding medium solved but 8 minutes over, narration absent; system design prompt wandered requirements-less for 15 minutes; behavioral answer 4 minutes of "we." Gap map allocated: 40% system design (the level-decider), 30% coding-under-conditions, 20% behavioral bank, 10% domain review. Weeks 1–3: daily 90-minute blocks interleaved — coding by pattern family with the redo-misses ritual (the miss log's top entry: sliding-window monotonicity checks — drilled); system design skeleton run 11 times against varied prompts, trade-off-stating practiced explicitly; story bank built one evening per week (9 stories, indexed; the "failure" family's story sourced from a postmortem the candidate had written — pre-quantified). Mocks at days 8 and 19: mock 1's recording exposed the silence habit (fixed by narrating even dead ends); mock 2's system design feedback ("you never asked about read/write ratio") patched the skeleton. Final week: taper — pattern list review, logistics, two early nights. Loop result: offer; the round-by-round notes recorded that the domain round asked exactly one deep question the 10% allocation had covered — and the retro file's existence made the NEXT loop's prep (a year later, different company) a two-week affair.

## Related skills
- `resume-tailoring` — upstream; the claims this prep must defend.
- `salary-negotiation` — downstream; the offer this prep produces.
- `two-pointer-sliding-window` / `graph-traversal-patterns` / `dynamic-programming-derivation` / `binary-search-variants` / `heap-and-priority-patterns` — the coding curriculum.
- `deep-work-scheduling` — the practice blocks' construction.
