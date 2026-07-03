---
name: research-question-framing
description: >
  Use when a research effort starts vague — narrowing "learn about X" into
  a question that's answerable, decision-connected, and scoped to the time
  available. Triggers: "research X for me", "we should look into",
  "what should the research question be", "this investigation is sprawling",
  "scope this research".
---

# Research Question Framing

## When to use this skill
- A research request arrives as a topic ("look into AI code review") instead of a question.
- An investigation is three weeks in with no convergence — usually a framing failure wearing an effort costume.
- NOT the research itself — this produces the question that `literature-review`, `survey-design`, or `experiment-design` then answer; ten minutes here saves days there.

## Prerequisites
- Access to whoever wants the research (or their written intent) — framing is negotiation with the requester, not solo wordsmithing.

## Workflow

1. **Extract the decision hiding behind the topic:** ask "what will you DO differently depending on what this finds?" until it lands on a choice, a budget, a design, or a bet ("look into AI code review" → the decision was "should we buy a tool this quarter, and which"). Research with no downstream decision is legitimate exactly once labeled as exploration with a timebox — the failure mode is decision-research funded as exploration and exploration expected to yield decisions.

2. **Decompose the topic into the question types it's hiding, and pick:** descriptive ("what's the current state of..."), comparative ("does A outperform B on..."), causal ("does X produce Y..."), predictive ("will Z hold at scale..."), or design ("how might we...") — each type demands different methods and evidence (`experiment-design` for causal, `survey-design` for descriptive prevalence, `literature-review` for state-of-knowledge). A vague topic is usually 4–5 questions of different types tangled; naming them separately is most of the untangling, and the decision from step 1 selects which ONE leads.

3. **Sharpen the chosen question until every noun is pinned:** population/context ("for teams like ours" → "for 20–50-eng teams on a monorepo"), the outcome variable and its measure ("better" → "review turnaround time and escaped-defect rate" — `metric-definition` discipline), the comparison baseline (against what?), and timeframe/conditions. The pin-test: could two people research this question independently and recognizably answer the SAME question? The unpinned version guarantees they won't.

4. **Check answerability against three gates:** evidence exists-or-is-generatable (some questions are cheap lookups, some need experiments you can run, some need data nobody has — know which BEFORE committing); the timebox fits the question's weight (a quarter-scale question asked of a two-day slot gets renegotiated: shrink the question or grow the slot, never silently deliver the quarter question badly in two days); and falsifiability-in-practice — what would a "no" look like, and would the requester accept it? (A question whose every answer routes to the same predetermined action is theater; surface that now, kindly.)

5. **Pre-state what would change the answer — the assumptions register:** the 2–4 load-bearing assumptions under the question ("assumes our review bottleneck is reviewer attention, not CI latency"), each one checkable early and cheaply. Half of derailed research is a false assumption discovered in week three that a day-one check would have caught; the register converts them into the first day's checklist and doubles as the scope fence (findings challenging an assumption pause the research for re-framing, rather than silently bending the question).

6. **Write the one-line brief and socialize it:** *question (pinned) + decision it feeds + timebox + evidence types expected + out-of-scope list.* The out-of-scope list (from step 2's unchosen questions — parked visibly, not lost) is the anti-sprawl mechanism: when week two's interesting tangent appears, it gets parked against the list instead of adopted by drift. Requester signs the brief — the cheap ceremony that prevents the week-four "that's not what I meant."

7. **Re-check the frame at the timebox's midpoint:** is the question still the right one given what's emerged? Legitimate re-framing is announced and re-signed (question v2, with the reason); illegitimate re-framing is the silent drift where the deliverable answers a different question than the brief. And when findings arrive: report against the ORIGINAL question first, tangents second, explicitly labeled — the discipline that keeps research trusted enough to keep getting funded.

## Common pitfalls
- Topic-shaped assignments accepted as-is: "research the competitive landscape" — three weeks of museum-building later, no decision moved (`competitive-analysis` has the same prerequisite for the same reason). Extract the decision or timebox the exploration, always.
- Boiling-the-ocean questions: "what's the best architecture?" — unpinned population, unpinned outcome, no baseline. The pin-test (step 3) fails; so will the research.
- The predetermined question: research commissioned to bless a decision already made — every finding routed to yes. Step 4's falsifiability gate names it early; delivering it anyway just spends credibility.
- Scope accretion by interestingness: each tangent adopted because it's genuinely interesting, deadline arrives with six half-answers. The out-of-scope list (step 6) is the parking lot that makes "not now" cheap.
- Assumption archaeology in week three: the whole question premised on a fact one phone call would have falsified on day one (step 5's register exists for exactly this).
- Framing solo and presenting the polished question to a requester who meant something else — framing is the negotiation; the sign-off (step 6) is its receipt.

## Example
Request from the VP: "look into whether we should be worried about our on-call load." Decision extraction (two questions, five minutes): the real fork was "do we fund a dedicated SRE hire next quarter, or fix rotation-side?" Decomposition surfaced four tangled questions — descriptive (how bad IS the load, measured?), comparative (vs industry?), causal (what's driving it?), design (what would reduce it?) — with the hire decision selecting the descriptive+causal pair. Pinned: "over the last 2 quarters, what is per-engineer pager load (pages/week, off-hours fraction) across our 3 rotations, and what share traces to the top-5 alert sources?" — answerable from existing PagerDuty data (gate 1 pass), one-week timebox (gate 2), and a "no problem here" answer was genuinely acceptable (gate 3). Assumptions register caught the big one on day one: "assumes pages ≈ load" — a rotation interview revealed HALF the burden was un-paged Slack escalations, re-framing (announced, v2) to include them. Delivered against the brief: load concentrated in one rotation, 61% traceable to two alert sources — the decision resolved as "fix alerting first (`alerting-design`), revisit the hire in Q2," and the three parked questions became two later briefs and one deliberate never.

## Related skills
- `literature-review` / `survey-design` / `experiment-design` — the methods the framed question dispatches to.
- `metric-definition` — pinning the outcome variables.
- `prd-writing` — the same problem-before-solution discipline, product-side.
- `note-synthesis` — keeping the investigation's findings claim-shaped.
