---
name: cash-flow-forecast
description: >
  Use when forecasting cash — 13-week direct forecasts, scenario cases,
  runway math, and the timing realities (collections, payroll) that P&L
  views hide. Triggers: "how much runway", "13-week cash flow", "will we
  make payroll", "cash forecast", "when do we run out", "scenario
  planning for cash". NOT for long-range P&L modeling (see
  financial-model-review for auditing those) — cash is a different
  animal from profit.
---

# Cash Flow Forecast

## Overview

Profit is an opinion; cash is a fact with a date — the profitable company that dies in week 7 because three customers pay in week 9 is this skill's founding case. The 13-week direct forecast at weekly grain, collections modeled from behavior not invoice terms, commitments sized against the downside case, and triggers pre-agreed before the bad week: that's the discipline.

## When to Use

- Runway questions with real stakes (fundraise timing, hiring pace, survival math).
- Building the rolling short-term cash discipline any finite-cash company needs.

**When NOT to use:**
- Long-range P&L models — different animal; `financial-model-review` audits those.

## Prerequisites

- Bank-account truth as the anchor, plus AR/AP agings, payroll schedule, and the contracts calendar (`invoice-expense-workflow`'s renewals feed).

## The Workflow

1. **Build the 13-week direct forecast as the workhorse:** weekly columns; rows = actual cash movements, not P&L lines — collections (not revenue), payroll by paydates, AP by when you'll actually pay, debt service, tax lumps, one-times. Direct method because the QUESTION is timing, and timing is what the direct view shows. Weekly grain is non-negotiable: the month that nets positive can contain nine insolvent days between payroll (week 2) and the big collection (week 4).

2. **Forecast collections from behavior, not invoice terms:** actual DSO by customer/segment (the net-30 base that pays at 47 — use 47), big customers (>10% of collections) forecast individually by their observed habits, payment seasonality (year-end AP freezes), pipeline revenue entering ONLY at closed + realistic lag. Optimistic collections are the single largest 13-week error — score yourself (step 6) and haircut.

3. **Time the outflows with their real calendar:** payroll on paydates (two-payroll vs three-payroll months matter at weekly grain), rent/debt on contract dates, tax lumps placed, renewals from the calendar — and the payment-timing LEVERS pre-identified (which AP can flex two weeks without damage, whose goodwill you'd spend). Knowing the levers BEFORE the tight week is the difference between managing cash and panicking at it.

4. **Run three scenarios with named assumptions, not a blessed number:** base, downside (the named risks land: the renewal slips, DSO stretches, the big deal pushes), upside. Each scenario's delta = 4–6 named events, not a mood dial. Runway and minimum-cash-week per scenario — and commitments (hires, leases) sized against DOWNSIDE cash, because commitments are made once and survived weekly.

5. **Define the trigger ladder — the forecast's alarm wiring:** minimum acceptable cash (payroll + buffer), and pre-agreed actions at thresholds crossed IN THE FORECAST, not the bank account (the forecast's whole point is acting on week 9's problem in week 2): downside <8 weeks → hiring freeze + collections sprint; base <13 weeks → financing conversations start (raises take 3–6 months; the trigger leads the need by that much). Agreed with leadership NOW — threshold-crossing weeks are the worst weeks for calm decisions.

6. **Reforecast weekly against actuals, and score yourself:** roll the window, reconcile per line, track accuracy — collections error especially (the systematic 8–10% optimism, once measured, becomes a haircut). The 20-minute weekly ritual IS the practice; a 13-week built once for a board meeting is a photograph.

7. **Report the story, not the spreadsheet:** the weekly note = current balance, minimum-cash week and its number (the headline: WHEN is the tightest week, how tight), runway per scenario, trigger status, deltas from last week with causes. One screen (`stakeholder-update` discipline); the model attached for the one reader who wants it.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The P&L shows profit — cash will follow" | Revenue isn't collections, expenses aren't payments, and growth EATS cash through the AR gap even when profitable. The direct view exists because 'will follow' has a date, and the date can be after payroll. |
| "Invoice terms are net-30 — model collections at day 30" | Your actual DSO is 47, and the 17-day systematic optimism is the genre's biggest error. Behavioral basis, per major customer, with the measured haircut. |
| "Monthly granularity is enough for a quarter view" | The month nets fine around nine insolvent mid-month days. Weekly grain is where payroll collisions and collection gaps become visible — which is the forecast's entire job. |
| "One realistic scenario beats scenario theater" | The single scenario is implicitly the optimistic one, and commitments get sized against it. Downside-case sizing (step 4) is what keeps the hires survivable when the renewal slips. |
| "We'll decide on triggers if things get tight" | Deciding thresholds while below them is duress negotiation. The pre-agreed ladder converts the bad week into execution; improvised, it's a standoff. |
| "We built the 13-week for the board — done" | Unreconciled forecasts stay wrong the same way forever. The weekly reforecast-and-score loop is what made month three's forecast trustworthy — and what the example's investor was actually buying. |

## Red Flags

- Runway quoted from the P&L model, no direct forecast anywhere.
- Collections modeled at invoice terms.
- Monthly columns on a survival question.
- One scenario; commitments sized against it.
- No trigger ladder; financing conversations starting inside the need window.
- Forecast never reconciled; accuracy unknown.

## Verification

- [ ] 13-week direct forecast live at weekly grain, anchored to bank balances — file linked.
- [ ] Collections on behavioral DSO with big customers individual — basis shown.
- [ ] Outflow calendar complete (paydates, lumps, renewals); flex levers pre-listed.
- [ ] Three scenarios with named assumption deltas; commitments sized against downside — noted.
- [ ] Trigger ladder signed by leadership with lead times matching raise timelines — doc linked.
- [ ] Weekly reforecast running; accuracy log with haircuts applied — history shown.
- [ ] Weekly one-screen note going out with min-cash week as the headline.

## Example

Series-A startup, 14 months of book runway "per the model," CFO builds the 13-week anyway. Week one's build surfaces three things the monthly model hid: a two-payroll vs three-payroll collision with the office deposit (min-cash week 6, uncomfortably thin); collections reality — the two enterprise customers (34% of AR) pay at day 52, not net-30; and the tax lump nobody had weekly-placed. Scenarios: downside (renewal slips + DSO stretches) showed 7 weeks to minimum — against which the planned 4 hires got resequenced to 2-now-2-later. Trigger ladder agreed: base <6 months starts the bridge conversation. Week 5's reforecast fired a real signal: Acme slipped again, min-week moved 6 → 5 — the pre-identified AP lever plus a collections call bridged it without drama, BECAUSE the levers were pre-mapped. Month-two accuracy finding: collections 8% optimistic → haircut applied. When the fundraise started (trigger-led, months before the need), the 13-week practice itself became diligence material — the investor's comment: "companies that can show me this usually survive."

## Related skills

- `budget-variance-analysis` — the accrual-side sibling; this is the cash truth.
- `financial-model-review` — auditing the long-range model this disciplines.
- `invoice-expense-workflow` — the AP/renewals calendar feeding the outflows.
- `capacity-planning` — the same headroom-and-triggers thinking, infrastructure edition.
