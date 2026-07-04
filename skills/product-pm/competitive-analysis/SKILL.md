---
name: competitive-analysis
description: >
  Use when analyzing competitors systematically — structured teardowns of
  jobs-served, pricing, positioning, and gaps, producing decisions
  instead of feature-matrix envy. Triggers: "competitive analysis",
  "teardown of competitor X", "how do we compare to", "competitor
  launched something", "market landscape for our roadmap". NOT for
  pricing your own product (see pricing-analysis, which consumes this
  output) or one-off sales battlecards (a byproduct, not the goal).
---

# Competitive Analysis

## Overview

Competitive analysis ends in four verbs — compete, counter-position, concede, ignore — or it was a museum of screenshots. Frame by customer jobs (which surfaces the spreadsheet and the status quo as competitors), tear down hands-on, read pricing as strategy, and mine the public exhaust for trajectory.

## When to Use

- Informing a roadmap/positioning decision with a structured read of the field.
- A competitor launch triggered a "should we respond?" scramble.

**When NOT to use:**
- Your own price-setting — `pricing-analysis`.

## Prerequisites

- The decision this analysis feeds, named ("do we build X?", "how do we position against Y?").
- Access for hands-on evaluation: trials/demos of the top competitors.

## The Workflow

1. **Frame by customer job, not by vendor list:** start from the jobs your customers hire products for, then map who serves each — which surfaces the real competitive set, including the non-obvious members: spreadsheets, internal tools, "do nothing," and the adjacent-category product eating the job sideways. The vendor-list framing misses most B2B markets' biggest competitor: the status quo.

2. **Do the hands-on teardown per top competitor (3–5, not 15):** run the trial as a real user with a real scenario — onboarding friction, the core workflow's actual quality, permission model, extensibility. Capture: where it's genuinely better (write these honestly — the analysis finding the competitor bad at everything is describing its author), where it's weak, and what its *product choices reveal about its strategy* (what they made easy = who they're for).

3. **Read pricing as strategy, not as numbers:** packaging tiers (what's gated where = whom they monetize), the pricing metric (per-seat vs usage = their scaling thesis), the entry tier's shape (their acquisition motion), enterprise gates (their upmarket ambition). A competitor's price page is their strategy document published voluntarily.

4. **Mine the public exhaust for trajectory:** changelog cadence (velocity and its aim), job postings (six ML-engineer reqs IS the roadmap), review-site one-star themes (their churn reasons = your wedge), case-study logos (actual segment vs claimed). Trajectory beats snapshot: where they're *investing* matters more than where they are.

5. **Synthesize into a positioning map, not a feature matrix:** 2–3 axes that matter to buyers (ease-of-adoption vs depth; self-serve vs sales-led), each player placed with evidence, white space named. The 200-row checkbox matrix is the genre's signature waste — buyers don't buy checkboxes, and it converts analysis into gap-filling fuel detached from your segment's jobs.

6. **Land on explicit strategic calls — the section that makes it analysis:** per finding: *compete* (must match — table stakes for our segment, evidence attached), *counter-position* (they can't follow without breaking their model — the most valuable category), *concede* (their strength, not our fight — said out loud so sales stops promising it), or *ignore* (doesn't touch our segment). Every call ties back to the prerequisite decision.

7. **Keep it alive cheaply, refresh on triggers:** a monitoring lane (changelog RSS, review alerts, posting checks — an hour monthly) rather than annual re-teardowns; full refresh on events (their major launch, funding, pricing change). Date everything visibly — a competitive doc without dates becomes confidently wrong within two quarters, and stale "facts" in sales decks lose deals embarrassingly.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The feature matrix shows exactly where we're behind" | And drives a roadmap of gap-filling that makes you their lagging copy, financed by your own analysis. Buyers hire for jobs; the positioning map (step 5) is what decisions read. |
| "Their marketing site tells us what the product does" | It tells you their aspiration. The trial run with a real scenario (step 2) is where 'slick demo' separates from 'broken third click' — and where their strategy leaks through their defaults. |
| "Our teardown shows we're better on every dimension" | Then the teardown is a mirror, and every salesperson who's lost a deal to them discounts the whole document. The genuinely-better list is the credibility anchor. |
| "They launched — we need a response this quarter" | Most launches don't touch your segment's jobs. The compete/counter/concede/ignore discipline is the antidote to roadmap whiplash; the example's IGNORE saved a quarter. |
| "We compete with the other five vendors — that's the set" | The biggest competitor in most B2B markets is the spreadsheet and 'do nothing.' Job-framing (step 1) finds the rivals the vendor list can't see. |
| "The analysis from last year is still mostly right" | 'Mostly' is where the pricing model they abandoned lives on in your sales deck. Dates on everything; the monthly monitoring hour beats the annual archaeology. |

## Red Flags

- A deliverable that's mostly a feature checkbox grid.
- No hands-on trial evidence anywhere.
- Zero entries in the "genuinely better" column.
- Roadmap items justified purely by "competitor has it."
- No compete/counter/concede/ignore calls — findings without verbs.
- Undated claims circulating in sales materials.

## Verification

- [ ] The feeding decision named up front — doc header.
- [ ] Job-map includes non-vendor competitors (status quo, spreadsheets) — shown.
- [ ] Hands-on teardown evidence per top competitor, incl. genuinely-better items — artifacts linked.
- [ ] Pricing read as strategy with the packaging inferences stated.
- [ ] Positioning map with evidence-placed players and named white space — attached.
- [ ] Every finding carries one of the four calls, tied to the decision; concedes communicated to sales — links.
- [ ] Everything dated; monitoring lane live with owners.

## Example

Decision at stake: "should we build workflow automation, where CompetitorX just launched?" Jobs framing first: our segment (ops teams ≤50 seats) hires us for reconciliation accuracy; automation serves an adjacent job they mostly solve with Zapier. Teardown of X's launch (trial, real scenario): genuinely slick builder (noted honestly), but gated to their enterprise tier and coupled to per-seat pricing. Exhaust: X's postings all enterprise-sales; their G2 one-star theme = "got expensive fast." Positioning map: X moving upmarket, away from our segment. Strategic calls: IGNORE the launch for the roadmap (adjacent job, wrong segment — evidence attached); COUNTER-POSITION on pricing in marketing ("no per-seat penalty" — X can't follow without repricing their base); CONCEDE enterprise workflow depth explicitly (sales told to stop competing for 500-seat automation RFPs — two doomed deals worth of effort saved per quarter). The roadmap slot the panic would have consumed went to the reconciliation work the churn data supported. Six months later X raised prices again; the monitoring lane caught it, marketing shipped the comparison update within the week.

## Related skills

- `pricing-analysis` — your-side pricing fed by step 3's reading.
- `prioritization-frameworks` — where compete-calls enter the roadmap contest.
- `roadmap-communication` — carrying concede/ignore decisions to sales without leaks.
- `user-interview-synthesis` — the customer-jobs evidence underneath step 1.
