---
name: kpi-reporting-pack
description: >
  Use when building the recurring metrics pack for leadership — monthly/
  quarterly ops reviews with trend, target, and narrative per KPI, designed
  to drive decisions instead of documenting the past. Triggers: "monthly
  business review deck", "KPI pack", "ops review format", "board metrics",
  "our MBR is 60 slides and decides nothing".
---

# KPI Reporting Pack

## When to use this skill
- Establishing or fixing a recurring business/ops review (MBR, QBR, board metrics section).
- The current pack is long, late, and decision-free.
- NOT for defining the metrics themselves (`metric-definition` — every KPI in the pack must have survived that skill first) or one-off project updates (`stakeholder-update`).

## Prerequisites
- The KPI set already defined with owners, targets, and canonical implementations (`metric-definition` step 6) — a pack built on contested definitions spends every review relitigating numerators.
- The review's decision-makers and cadence named — the pack is built backward from what THEY must decide.

## Workflow

1. **Cap the pack at the decision-driving few:** 8–15 KPIs for a company-level review (per-function reviews get their own, smaller sets) — one page per KPI or tighter. The selection test per metric: "what decision changes when this moves?" (`metric-definition`'s prerequisite, enforced at pack level). Everything informational-but-not-decisional moves to an appendix or a self-serve dashboard link; the 60-slide pack isn't thorough, it's unprioritized (`prioritization-frameworks` step 4's everything-is-P0, reporting edition).

2. **Standardize the per-KPI page — same shape every metric, every month:**
   - **The number + target + variance** (RAG-colored against pre-agreed thresholds, not presenter mood).
   - **Trend:** 12+ months, not two points ("up from last month" hides the yearly slide; seasonality needs a year to see).
   - **Driver decomposition** where the metric has one (the `budget-variance-analysis` step 3 volume/rate habit, generalized).
   - **Narrative: 3 lines max** — cause, action, expected effect ("churn +0.4pt: concentrated in SMB monthly plans post-price-change; win-back campaign live; expect recovery by M+2" — never the restatement genre: "churn increased due to more customers leaving").
   - **Owner named on the page.**
   The fixed format is the feature: readers learn where to look, deviations pop, and the pack assembles from owners in parallel.

3. **Pair every target metric with its guardrail on the same page:** bookings WITH 90-day retention, deploy frequency WITH change-failure rate, tickets-closed WITH reopen rate (`metric-definition` step 4's pairs, made structural) — the pack that reports the target without the guardrail is publishing the incentive to game it.

4. **Automate the numbers, humanize the narrative:** figures pulled from the canonical metrics layer (never hand-keyed into slides — the transcription typo that misinforms a board is a real and recurring event; and hand-built packs arrive late, which kills the review cadence), owners writing ONLY the 3-line narratives. Target assembly cost: hours, not days. A pack too expensive to produce monthly becomes quarterly, then occasional, then archaeology.

5. **Run the review meeting on exceptions, not recitation:** greens acknowledged in one pass (nobody reads 12 green pages aloud), the meeting's time spent on reds/yellows and trend-breaks — each ending in a decision or an owned action with a date (captured per `meeting-notes-actions`; reviewed at the next pack's opening: "last month's actions" is page one). The pack circulated 24–48h ahead with pre-reads expected; the meeting is for deciding, not for encountering the numbers.

6. **Keep the definitions stable and the changes loud:** metric definitions versioned (`metric-definition` step 6 — a silent redefinition poisons every trend line in the pack); restatements flagged on the page when late data revises history ("Jan restated +2%: late enterprise invoices"); and target changes are decisions recorded in the pack, never quiet edits (the target that moves to meet the number inverts the entire instrument).

7. **Audit the pack itself twice a year:** which pages drove a decision this half? (none = candidate for the appendix); which decisions got made WITHOUT the pack's data (= the missing page); is the RAG honest (a pack that's been all-green through a rough half has a threshold problem or a courage problem — `stakeholder-update`'s watermelon, institutionalized); and prune — packs only grow unless someone owns the shrinking (the same entropy as `design-system-tokens` step 7, reporting edition).

## Common pitfalls
- The recitation meeting: 45 minutes reading numbers everyone could read, 5 minutes of rushed discussion on the one red — exception-based running order (step 5) inverts it.
- Two-point "trends": up-from-last-month on a metric that's down 30% year-over-year — the 12-month line is non-negotiable (step 2).
- Narrative restatement: "revenue was below target due to lower sales" — the null explanation again (`budget-variance-analysis`'s same banned genre); cause-action-expectation or blank.
- Hand-keyed numbers: the deck's churn figure disagreeing with the dashboard's mid-meeting — credibility spent on a copy-paste (step 4's automation).
- Metric creep: every function adding "just one" page until the pack is 60 slides and the review is a tour — the twice-yearly prune (step 7) with the decision-test.
- All-green theater: thresholds set generous, narratives sanded smooth, leadership "informed" into a surprise — the RAG honesty audit exists because packs drift toward comfort (step 7).

## Example
Company MBR: 58 slides, assembled over 4 days, meeting a recitation, three months since any page changed a decision. Rebuild: decision-test cut the pack to 11 KPIs (the other ~40 metrics to a linked self-serve dashboard — twice-visited pages, it turned out); per-KPI template enforced with 12-month trends and guardrail pairs (bookings gained its NRR companion; the support CSAT page gained reopen-rate — which immediately exposed that a "great" CSAT quarter had been bought with premature ticket closes); numbers wired from the dbt metrics layer (assembly: 4 days → 3 hours, and the hand-key typo class died); narratives capped at 3 lines with the restatement genre banned by example. Meeting rebuilt: pre-read expected, page one = last month's actions, agenda = the 3 non-greens. Month two's review produced the format's proof: the SMB churn page's driver split (plan-type decomposition) turned a 40-minute anecdote exchange into a 10-minute decision (pause the price rollout for monthly plans — the action, owner, and date in the notes). The six-month audit cut two pages nobody had decided from and added one (pipeline coverage — the metric behind two decisions made off-pack). Pack: 11 pages, 90-minute review, and the all-green problem hasn't recurred since reopen-rate made honesty structural.

## Related skills
- `metric-definition` — the rigor every page stands on.
- `budget-variance-analysis` — the finance section's driver discipline.
- `stakeholder-update` — the single-project sibling.
- `meeting-notes-actions` — the action capture the review runs on.
