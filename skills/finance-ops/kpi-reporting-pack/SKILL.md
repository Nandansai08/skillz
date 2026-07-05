---
name: kpi-reporting-pack
description: >
  Use when building the recurring metrics pack for leadership — monthly/
  quarterly ops reviews with trend, target, and narrative per KPI,
  designed to drive decisions instead of documenting the past. Triggers:
  "monthly business review deck", "KPI pack", "ops review format",
  "board metrics", "our MBR is 60 slides and decides nothing". NOT for
  defining the metrics themselves (see metric-definition) or one-off
  project updates (see stakeholder-update).
---

# KPI Reporting Pack

## Overview

A pack earns its meeting by the decision-test: what changes when this metric moves? Eight to fifteen KPI pages — number, 12-month trend, driver split, three-line narrative, guardrail pair — reviewed on exceptions, with numbers automated so the humans spend their time on the reds. The 60-slide pack isn't thorough; it's unprioritized.

## When to Use

- Establishing or fixing a recurring business/ops review (MBR, QBR, board metrics).
- The current pack is long, late, and decision-free.

**When NOT to use:**
- Metric definitions — `metric-definition`, first.
- One-off project status — `stakeholder-update`.

## Prerequisites

- The KPI set defined with owners, targets, canonical implementations (`metric-definition` step 6) — a pack on contested definitions relitigates numerators monthly.
- The review's decision-makers and cadence named.

## The Workflow

1. **Cap the pack at the decision-driving few:** 8–15 KPIs for a company review. The selection test per metric: "what decision changes when this moves?" Everything informational goes to an appendix or self-serve dashboard link.

2. **Standardize the per-KPI page — same shape every metric, every month:**
   - **Number + target + variance** (RAG against pre-agreed thresholds, not presenter mood).
   - **Trend:** 12+ months ("up from last month" hides the yearly slide; seasonality needs a year).
   - **Driver decomposition** where one exists (`budget-variance-analysis` step 3's habit, generalized).
   - **Narrative: 3 lines max** — cause, action, expected effect. Never the restatement genre ("churn increased due to more customers leaving").
   - **Owner named on the page.**
   The fixed format IS the feature: readers learn where to look, deviations pop, owners assemble in parallel.

3. **Pair every target metric with its guardrail on the same page:** bookings WITH retention, deploy frequency WITH change-failure rate, tickets-closed WITH reopen rate (`metric-definition` step 4's pairs, made structural) — the pack reporting the target without the guardrail is publishing the incentive to game it.

4. **Automate the numbers, humanize the narrative:** figures from the canonical metrics layer, never hand-keyed (the transcription typo that misinforms a board is a recurring real event; and hand-built packs arrive late, which kills the cadence); owners write ONLY the 3-line narratives. Assembly cost target: hours. A pack too expensive to produce monthly becomes quarterly, then occasional, then archaeology.

5. **Run the meeting on exceptions, not recitation:** greens acknowledged in one pass; the time spent on reds/yellows and trend-breaks — each ending in a decision or an owned action with a date (captured per `meeting-notes-actions`; "last month's actions" is page one). Pack circulated 24–48h ahead; the meeting is for deciding, not encountering.

6. **Keep definitions stable and changes loud:** definitions versioned (silent redefinitions poison every trend line); restatements flagged on the page; target changes are recorded decisions, never quiet edits (the target that moves to meet the number inverts the instrument).

7. **Audit the pack itself twice a year:** which pages drove a decision this half? (None = appendix candidate.) Which decisions got made WITHOUT the pack's data? (= the missing page.) Is the RAG honest? (All-green through a rough half = threshold problem or courage problem.) And prune — packs only grow unless someone owns the shrinking.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Every team needs their page — it's only fair" | Fairness-driven inclusion built the 58-slide pack that decided nothing. The decision-test is the filter; team visibility is what dashboards are for. |
| "Up 3% from last month — trend included" | Two points hide the 30% yearly slide and every seasonal pattern. Twelve months minimum, or the 'trend' is a mood. |
| "The narrative explains the number: churn rose because more customers left" | The null explanation again. Cause-action-expectation in three lines, or leave the box empty — empty is at least honest about what's known. |
| "I'll paste the numbers in tonight — faster than wiring the data" | Tonight's paste is next month's board-visible typo, and the hand-built pack drifts late until the cadence dies. Automation is what makes monthly survivable. |
| "Let's walk through each slide for completeness" | The recitation meeting: 45 minutes reading aloud, 5 on the one red. Exceptions-based running order is the difference between a review and a tour. |
| "Adjust the target — the market shifted" | Maybe true — as a RECORDED decision with a reason. The quietly-moved target converts the pack from instrument to mirror, and every future green inherits the suspicion. |

## Red Flags

- Pack length growing quarter over quarter.
- Numbers hand-keyed; dashboard and deck disagreeing mid-meeting.
- Targets without guardrails anywhere.
- All-green packs through visibly rough periods.
- No "last month's actions" page.
- The same pack circulating for years with no page ever retired.

## Verification

- [ ] KPI count 8–15; each page passes the decision-test — the cut list from the last audit shown.
- [ ] Per-page template enforced (number/target/RAG, 12-mo trend, drivers, 3-line narrative, owner) — spot-check three pages.
- [ ] Guardrail pairs on every target page — verified.
- [ ] Numbers pulled from the metrics layer — pipeline linked; assembly time measured in hours.
- [ ] Meeting runs exceptions-first with actions captured and reviewed next cycle — notes linked.
- [ ] Definition/target changes versioned and visible — change log.
- [ ] Semi-annual pack audit done — pages cut/added with reasons.

## Example

Company MBR: 58 slides, assembled over 4 days, meeting a recitation, three months since any page changed a decision. Rebuild: decision-test cut the pack to 11 KPIs (the other ~40 metrics to a linked dashboard — twice-visited, it turned out); per-KPI template with 12-month trends and guardrail pairs (bookings gained NRR; support CSAT gained reopen-rate — which immediately exposed that a "great" CSAT quarter had been bought with premature closes); numbers wired from the dbt metrics layer (assembly: 4 days → 3 hours, and the hand-key typo class died). Meeting rebuilt: pre-read expected, page one = last month's actions, agenda = the 3 non-greens. Month two's proof: the SMB churn page's driver split turned a 40-minute anecdote exchange into a 10-minute decision (pause the price rollout for monthly plans — action, owner, date). The six-month audit cut two pages nobody had decided from and added one (pipeline coverage — behind two decisions made off-pack). Pack: 11 pages, 90-minute review, and the all-green problem hasn't recurred since reopen-rate made honesty structural.

## Related skills

- `metric-definition` — the rigor every page stands on.
- `budget-variance-analysis` — the finance section's driver discipline.
- `stakeholder-update` — the single-project sibling.
- `meeting-notes-actions` — the action capture the review runs on.
