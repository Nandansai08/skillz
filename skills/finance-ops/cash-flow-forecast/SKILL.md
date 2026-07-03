---
name: cash-flow-forecast
description: >
  Use when forecasting cash — 13-week direct forecasts, scenario cases,
  runway math, and the timing realities (collections, payroll) that P&L
  views hide. Triggers: "how much runway", "13-week cash flow", "will we
  make payroll", "cash forecast", "when do we run out", "scenario planning
  for cash".
---

# Cash Flow Forecast

## When to use this skill
- Runway questions with real stakes (fundraise timing, hiring pace, survival math).
- Building the rolling short-term cash discipline any finite-cash company needs.
- NOT for long-range P&L modeling (`financial-model-review` audits those) — cash is a different animal from profit, and conflating them is this domain's original sin.

## Prerequisites
- Bank-account truth as the anchor (forecasts start from actual balances, not book balances), plus AR/AP agings, payroll schedule, and the contracts calendar (renewals, debt payments — `invoice-expense-workflow`'s calendar feeds this).

## Workflow

1. **Build the 13-week direct forecast as the workhorse:** weekly columns (13 weeks ≈ one quarter — long enough to see trouble, short enough to be forecastable); rows = actual cash movements, not P&L lines — collections (not revenue), payroll by actual paydates, AP payments by when you'll actually pay, debt service, taxes (the quarterly lumps that ambush), one-times. Direct method (money in, money out) beats indirect (profit + adjustments) at this range because the QUESTION is timing, and timing is exactly what the direct view shows: the profitable company that dies in week 7 because three customers pay in week 9 is the case this forecast exists to catch.

2. **Forecast collections from behavior, not invoice terms:** actual DSO by customer/segment (the net-30 base that pays in 47 — use the 47), the big-customer watch (any customer >10% of collections gets forecast individually, by their observed habits), seasonality of payments (year-end AP freezes at your enterprise customers), and pipeline revenue entering ONLY at closed + realistic collection lag. The single largest forecast error in most 13-weeks is optimistic collections — score your own accuracy (step 6) and haircut accordingly.

3. **Time the outflows with their real calendar:** payroll on paydates (the two-payroll month vs three-payroll month difference matters at the weekly grain), rent/debt on contract dates, the tax lumps, annual renewals from the contracts calendar, and the payment-timing LEVERS pre-identified (which AP can flex two weeks without damage, which vendors' goodwill you'd spend — knowing the levers BEFORE the tight week is the difference between managing cash and panicking at it; `vendor-evaluation`'s relationship capital is what these levers draw on).

4. **Run three scenarios with named assumptions, not a blessed number:** base (current trajectory, honest collections), downside (the named risks land: the big renewal slips, collections stretch 2 weeks, the pipeline's biggest deal pushes a quarter), upside (mostly for capacity decisions). Each scenario's assumptions LISTED (the delta between scenarios is 4–6 named events, not a mood dial at ±15%); runway and minimum-cash-week computed per scenario. The downside case is the decision-grade one: commitments (hires, leases, contracts) get sized against DOWNSIDE cash, because commitments are made once and survived weekly.

5. **Define the trigger ladder — the forecast's alarm wiring:** minimum acceptable cash (payroll + a buffer — the floor below which nothing goes), and pre-agreed actions at thresholds crossed IN THE FORECAST (not in the bank account — the forecast's whole point is acting on week 9's problem in week 2): e.g., downside shows <8 weeks runway → hiring freeze + collections sprint; <13 weeks in base → financing conversations start (fundraises take 3–6 months; the trigger must lead the need by that much — `slo-definition`'s error-budget-policy logic, applied to survival). Triggers agreed with leadership NOW, because threshold-crossing weeks are the worst weeks for calm decisions.

6. **Reforecast weekly against actuals, and score yourself:** roll the window (drop the closed week, add week 14), reconcile forecast-vs-actual per line, and track the accuracy trend — collections forecast error especially (the systematic 10% optimism, once measured, becomes a haircut — `budget-variance-analysis` step 7's recurring-pattern rule at weekly cadence). The 20-minute weekly ritual is the practice; a 13-week built once for a board meeting is a photograph, not a forecast.

7. **Report the story, not the spreadsheet:** the weekly cash note = current balance, minimum-cash week and its number (the forecast's headline: WHEN is the tightest week and how tight), runway per scenario, trigger status, and the deltas from last week with causes ("min-week moved from wk9 to wk7: Acme's payment slipped — collections call scheduled"). One screen (`stakeholder-update` discipline); the full model attached for the one reader who wants it.

## Common pitfalls
- Profit-cash conflation: the P&L says fine, the bank account disagrees — revenue isn't collections, expenses aren't payments, and growth EATS cash through working capital even when profitable (the faster you grow, the wider the AR gap — the growth-kills-profitable-company mechanism, and it's a timing story only the direct view shows).
- Invoice-term collections: forecasting net-30 as day-30 arrival while your actual DSO is 47 — the systematic optimism that makes week-9 surprises (step 2's behavioral basis).
- Monthly grain hiding weekly death: the month nets positive; payroll is week 2 and the big collection is week 4 — fine at monthly resolution, insolvent for nine days in the middle. Weekly grain for the 13-week, non-negotiable.
- The single blessed scenario: one forecast, implicitly the optimistic one, commitments sized against it — the downside case (step 4) is what commitments must survive.
- Triggers reverse-engineered in the bad week: deciding hiring-freeze thresholds WHILE below them — the pre-agreement (step 5) is the entire value; thresholds negotiated under duress are theater.
- Forecast-as-artifact: built for the board deck, never reconciled, wheeled out quarterly — unscored forecasts stay wrong the same way forever (step 6's accuracy loop is what makes month three's forecast better than month one's).

## Example
Series-A startup, 14 months of book runway "per the model," CFO builds the 13-week anyway. Week one's build surfaces three things the monthly model hid: a two-payroll vs three-payroll month collision with the office deposit (min-cash week 6, uncomfortably thin); collections reality — the two enterprise customers (34% of AR) actually pay at day 52, not net-30 (behavioral basis adopted, forecast re-cut); and the tax lump nobody had weekly-placed. Scenarios: downside (bigger renewal slips a quarter + DSO stretches) showed 7 weeks to minimum cash — against which the planned 4 hires got resequenced to 2-now-2-later (sized against downside, per step 4). Trigger ladder agreed: base-runway <6 months starts the bridge conversation. Week 5's reforecast fired a real signal: Acme's payment slipped again, min-week moved from 6 to 5 — the pre-identified AP lever (two vendors, two weeks, relationship-safe) plus a collections call bridged it without drama, BECAUSE the levers were pre-mapped. The accuracy log's month-two finding: collections systematically 8% optimistic → haircut applied. When the fundraise started (trigger-led, months before the need), the 13-week practice itself became diligence material — the investor's comment: "companies that can show me this usually survive."

## Related skills
- `budget-variance-analysis` — the accrual-side sibling; this is the cash truth.
- `financial-model-review` — auditing the long-range model this short-range one disciplines.
- `invoice-expense-workflow` — the AP/renewals calendar feeding the outflows.
- `capacity-planning` — the same headroom-and-triggers thinking, infrastructure edition.
