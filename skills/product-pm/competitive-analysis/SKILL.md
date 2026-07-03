---
name: competitive-analysis
description: >
  Use when analyzing competitors systematically — structured teardowns of
  jobs-served, pricing, positioning, and gaps, producing decisions instead
  of feature-matrix envy. Triggers: "competitive analysis", "teardown of
  competitor X", "how do we compare to", "competitor launched something",
  "market landscape for our roadmap".
---

# Competitive Analysis

## When to use this skill
- Informing a roadmap/positioning decision with a structured read of the field.
- A competitor launch triggered a "should we respond?" scramble.
- NOT for pricing your own product (that's `pricing-analysis`, which consumes this skill's output) or one-off sales battlecards (a byproduct here, not the goal).

## Prerequisites
- The decision this analysis feeds, named ("do we build X?", "how do we position against Y's launch?") — analysis without a pending decision becomes a museum of screenshots.
- Access for hands-on evaluation: trials/demos of the top competitors (reading marketing pages is not analysis).

## Workflow

1. **Frame by customer job, not by vendor list:** start from the jobs your customers hire products for, then map who serves each job — which surfaces the real competitive set, including the non-obvious members: spreadsheets, internal tools, "do nothing," and the adjacent-category product eating the job sideways. The vendor-list framing misses the biggest competitor in most B2B markets, which is the status quo.

2. **Do the hands-on teardown per top competitor (3–5, not 15):** run the trial as a real user with a real scenario — onboarding friction, the core workflow's actual quality (not its screenshot quality), collaboration/permission model, extensibility. Capture: where it's genuinely better (write these honestly — the analysis that finds the competitor bad at everything is describing its author), where it's weak, and what its *product choices reveal about its strategy* (what they made easy = who they're for).

3. **Read pricing as strategy, not as numbers:** packaging tiers (what's gated where = which customer they monetize), the pricing metric (per-seat vs usage vs flat — reveals their scaling thesis and their customers' cost anxiety), the free/entry tier's shape (their acquisition motion), enterprise-tier gates (their upmarket ambition). A competitor's price page is their strategy document published voluntarily (`pricing-analysis` consumes this reading for your own decisions).

4. **Mine the public exhaust for trajectory:** changelogs/release cadence (velocity and where it's aimed), job postings (a competitor hiring 6 ML engineers is telling you the roadmap), review-site complaints (G2/Capterra one-star themes = their churn reasons = your wedge), community/forum tone, and case-study customer logos (their actual segment vs their claimed one). Trajectory beats snapshot: where they're *investing* matters more than where they are.

5. **Synthesize into a positioning map, not a feature matrix:** the deliverable's core is 2–3 axes that matter to buyers (e.g., ease-of-adoption vs depth; SMB-self-serve vs enterprise-sales-led), each player placed with evidence, and the white space named. The 200-row feature-checkbox matrix is the genre's signature waste — buyers don't buy checkboxes, and it converts analysis into feature-envy fuel ("they have X, we need X") detached from whether X serves your segment's jobs.

6. **Land on explicit strategic calls — the section that makes it analysis:** for each finding: *compete* (we must match this — it's table stakes for our segment, with evidence), *counter-position* (they can't follow us here without breaking their model — the most valuable category, e.g. their per-seat pricing can't chase usage-based buyers), *concede* (their strength, not our fight — say it out loud so sales stops promising it), or *ignore* (their move doesn't touch our segment). Every recommendation ties back to the prerequisite decision, and disagreements about the calls are strategy conversations working as intended.

7. **Keep it alive cheaply, refresh on triggers:** a monitoring lane (changelog RSS, review alerts, job-posting checks — an hour monthly) rather than annual re-teardowns; full refresh triggered by events (their major launch, funding, pricing change, your segment expansion). Date everything visibly — a competitive doc without dates becomes confidently wrong within two quarters, and stale competitive "facts" in sales decks are how deals get lost embarrassingly.

## Common pitfalls
- Feature-matrix myopia: 40 rows of checkboxes driving a roadmap of gap-filling — you become their lagging copy, financed by your own analysis. Jobs and positioning axes, not checkboxes (step 5).
- Marketing-page analysis: cataloging claims without touching the product — their marketing describes their aspiration, not their product; the trial run (step 2) is non-negotiable.
- Strawman teardowns that flatter your product on every axis — comforting, useless, and instantly discounted by every salesperson who's lost a deal to the "inferior" competitor. The genuinely-better list (step 2) is the credibility anchor.
- Panic-responding to every competitor launch — most launches don't touch your segment's jobs; the compete/counter/concede/ignore discipline (step 6) is the antidote to roadmap whiplash (`prioritization-frameworks` step 7's triage lane, competitive edition).
- Ignoring the status-quo competitor: your biggest rival is the spreadsheet, and no feature matrix contains it (step 1).
- One-and-done: the analysis from 14 months ago still circulating in sales decks, describing a pricing model the competitor abandoned. Dates + triggers (step 7).

## Example
Decision at stake: "should we build workflow automation, where CompetitorX just launched?" Jobs framing first: our segment (ops teams ≤50 seats) hires us for reconciliation accuracy; automation serves an adjacent job they mostly solve with Zapier. Teardown of X's launch (trial, real scenario): genuinely slick builder (noted honestly), but gated to their enterprise tier and coupled to their per-seat model. Exhaust: X's job postings all enterprise-sales; their G2 one-star theme = "got expensive fast." Positioning map: X moving upmarket, away from our segment. Strategic calls: IGNORE the automation launch for the roadmap (adjacent job, wrong segment — with the evidence attached); COUNTER-POSITION on pricing in marketing ("no per-seat penalty" messaging — X can't follow without repricing their base); CONCEDE enterprise workflow depth explicitly (sales told to stop competing for 500-seat automation RFPs — two doomed deals worth of effort saved per quarter). The roadmap slot the panic would have consumed went to the reconciliation work the churn data actually supported. Six months later X raised prices again; the monitoring lane caught it, marketing shipped the comparison update within the week.

## Related skills
- `pricing-analysis` — your-side pricing decisions fed by step 3's reading.
- `prioritization-frameworks` — where compete-calls enter the roadmap contest.
- `roadmap-communication` — carrying concede/ignore decisions to sales without leaks.
- `user-interview-synthesis` — the customer-jobs evidence underneath step 1.
