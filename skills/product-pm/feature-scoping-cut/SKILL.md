---
name: feature-scoping-cut
description: >
  Use when a feature won't fit its timeline and scope must shrink without
  gutting the value — finding the kernel, choosing cuts that preserve the
  outcome, cutting honestly vs cutting corners. Triggers: "descope this",
  "won't make the deadline", "what can we cut", "MVP version of this",
  "feature is too big for the quarter".
---

# Feature Scoping Cut

## When to use this skill
- Estimate exceeds the slot: the feature as specced won't land by the date that matters.
- An "MVP" discussion is drifting toward either everything-anyway or something-useless.
- NOT for slicing work into shippable increments of the SAME scope — that's `user-story-slicing`; cutting changes what ships, slicing changes when.

## Prerequisites
- The feature's problem statement and success metric (`prd-writing` steps 1, 3) — you cannot protect the kernel without knowing what the kernel is FOR.
- A real constraint (date, capacity) — phantom deadlines produce phantom cuts that get argued back in.

## Workflow

1. **Restate the kernel: the smallest thing that delivers the core outcome to the core user.** One sentence, derived from the success metric: "ops manager reconciles a day's payouts without a spreadsheet." Everything in the spec is now judged as kernel / amplifier / barnacle. If the team can't agree on the kernel sentence, the scoping fight is actually a strategy fight — settle that first (`prd-writing`'s problem statement is the tiebreaker).

2. **Cut along the standard axes, in this order of preference:**
   - **Audience:** ship for the primary segment only (one plan tier, one region, admins-only). Cleanest cut — full experience, narrower door.
   - **Input/data variety:** CSV before Excel, one currency, files ≤50MB with an honest error. Variety is where effort hides.
   - **Polish and automation:** manual-behind-the-scenes for rare paths ("concierge" operations: the export edge case handled by support for the first month), batch instead of real-time, good-enough empty states.
   - **Adjacent conveniences:** the settings page, the bulk action, the second entry point — amplifiers by definition.
   What you do NOT cut: the kernel's error handling where failure loses data/money (`user-story-slicing`'s same caveat), security, and the instrumentation that measures success — a feature that ships without its metric events cannot be judged, which converts the cut from "v1" into "forever."

3. **Distinguish cut-for-later from cut-forever, out loud:** each cut gets a disposition — deferred (ticketed, revisit trigger named: "Excel support: revisit if >20% of upload errors are xlsx") or dropped (with the reason). The undifferentiated "phase 2" pile is where credibility goes to die, because everyone knows phase 2 never comes; named triggers make deferral honest.

4. **Check the cut set against coherence, not just size:** the shipped subset must make sense as a product someone uses start-to-finish — cutting the middle of a workflow ("upload works, mapping works, but errors just... fail") saves effort and ships nothing usable. Walk the primary user's complete path through the reduced scope; every gap is either restored, concierge'd, or blocked with honest messaging ("Excel coming — convert to CSV for now" beats a crash).

5. **Re-estimate the reduced scope with the builders, and bank margin:** cuts that "should" halve the estimate often don't (the kernel contained the hard part all along — which is information: the deadline conversation changes character if cutting 60% of scope saves 20% of time). Target landing at ~80% of the slot; a cut that exactly fills the timeline has pre-spent the buffer that reality always invoices.
 
6. **Communicate the cut as a decision, not an apology:** stakeholders hear WHAT ships, for whom, when; what's deferred with its trigger; what's dropped with its reason (`stakeholder-update` carries it; `roadmap-communication` step 6 if the roadmap shifted). The sales-facing version needs explicit "don't promise the deferred list" markings — a cut communicated vaguely gets un-cut by promise.

7. **After ship, judge the cuts with the metric:** the success measure (protected in step 2) now arbitrates — did the kernel deliver the outcome? Which deferred items' triggers actually fired? (Usually fewer than half — the quiet vindication of cutting.) Which cut generated real pain? (Restore it with evidence, ahead of the speculative rest.) This loop is what makes the NEXT scoping conversation fast and trusted.

## Common pitfalls
- Cutting quality instead of scope: keeping every feature but skipping tests, error handling, and review — the invisible cut that ships the whole surface area rotten. Cut visible scope; keep the kernel solid.
- The MVP that's minimum without being viable: so much cut that it can't complete anyone's real task — validating nothing except that broken things don't retain users (step 4's coherence walk is the guard).
- Cutting the instrumentation "temporarily" — now the feature can't prove itself, the deferred list can't be prioritized with data, and the retro is vibes (step 2's do-not-cut list).
- Salami re-scoping: five small "just this one addition" restorations that silently rebuild the original scope by week 4. The disposition list (step 3) is the contract; additions reopen it explicitly.
- Protecting a pet amplifier by relabeling it kernel — the test: does the success metric move without it? If yes, it's an amplifier, whoever loves it.
- Cutting by effort alone ("drop the expensive parts") — sometimes the expensive part IS the kernel, and the honest move is moving the date, not shipping the cheap husk.

## Example
Feature: payout reconciliation (from the `prd-writing` example), specced at 9 weeks against a 6-week slot. Kernel sentence: "ops manager reconciles one day's payouts without a spreadsheet." Cuts: audience → US-entity accounts only (EU tax formats deferred, trigger: "3+ EU enterprise requests" — it had been the estimate's biggest line); variety → CSV export only (the in-app view deferred); automation → refund-adjustment edge case handled by support macro for launch month (concierge — 40 est. cases/month, 5 min each); the settings page dropped outright. NOT cut: per-row error reporting (data-integrity path) and the reconciliation-time instrumentation. Re-estimate: 5.5 weeks — the margin absorbed a real one-week auth surprise. Post-ship: success metric hit at week 7; of four deferred items, one trigger fired (EU requests — built next quarter with evidence in hand), the in-app view's never did (support macro logs showed 11 cases/month, not 40 — deferred again, cheaply, with data). The step-7 retro took twenty minutes because every cut had a written disposition to check.

## Related skills
- `prd-writing` — the kernel's source of truth.
- `user-story-slicing` — sequencing what survives the cut.
- `roadmap-communication` — carrying the deferral news outward.
- `prioritization-frameworks` — where the deferred list re-competes.
