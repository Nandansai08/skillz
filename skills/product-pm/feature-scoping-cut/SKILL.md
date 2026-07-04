---
name: feature-scoping-cut
description: >
  Use when a feature won't fit its timeline and scope must shrink without
  gutting the value — finding the kernel, choosing cuts that preserve the
  outcome, cutting honestly vs cutting corners. Triggers: "descope this",
  "won't make the deadline", "what can we cut", "MVP version of this",
  "feature is too big for the quarter". NOT for slicing work into
  shippable increments of the SAME scope (see user-story-slicing) —
  cutting changes what ships; slicing changes when.
---

# Feature Scoping Cut

## Overview

Scope cutting protects the kernel — the smallest thing delivering the core outcome to the core user — and cuts along audience, variety, polish, and convenience axes, never along quality. Every cut gets a disposition (deferred-with-trigger or dropped-with-reason), because the undifferentiated "phase 2" pile is where credibility dies.

## When to Use

- Estimate exceeds the slot: the feature as specced won't land by the date that matters.
- An "MVP" discussion is drifting toward either everything-anyway or something-useless.

**When NOT to use:**
- Sequencing the same scope into increments — `user-story-slicing`.

## Prerequisites

- The feature's problem statement and success metric (`prd-writing`) — you cannot protect the kernel without knowing what it's FOR.
- A real constraint (date, capacity) — phantom deadlines produce phantom cuts that get argued back in.

## The Workflow

1. **Restate the kernel: the smallest thing that delivers the core outcome to the core user.** One sentence, derived from the success metric: "ops manager reconciles a day's payouts without a spreadsheet." Everything in the spec is now kernel / amplifier / barnacle. If the team can't agree on the kernel sentence, the scoping fight is actually a strategy fight — settle that first.

2. **Cut along the standard axes, in order of preference:**
   - **Audience:** primary segment only (one plan tier, one region, admins-only). Cleanest cut — full experience, narrower door.
   - **Input/data variety:** CSV before Excel, one currency, files ≤50MB with an honest error.
   - **Polish and automation:** manual-behind-the-scenes for rare paths (concierge ops for the first month), batch instead of real-time.
   - **Adjacent conveniences:** the settings page, the bulk action, the second entry point.
   What you do NOT cut: the kernel's error handling where failure loses data/money, security, and the instrumentation that measures success — a feature shipping without its metric events cannot be judged, which converts the cut from "v1" into "forever."

3. **Distinguish cut-for-later from cut-forever, out loud:** each cut gets a disposition — deferred (ticketed, revisit trigger named: "Excel support: revisit if >20% of upload errors are xlsx") or dropped (with the reason). Named triggers make deferral honest; everyone knows "phase 2" never comes.

4. **Check the cut set against coherence, not just size:** the shipped subset must make sense start-to-finish — cutting the middle of a workflow saves effort and ships nothing usable. Walk the primary user's complete path through the reduced scope; every gap is restored, concierge'd, or blocked with honest messaging ("Excel coming — convert to CSV for now" beats a crash).

5. **Re-estimate the reduced scope with the builders, and bank margin:** cuts that "should" halve the estimate often don't (the kernel contained the hard part — which is information: the deadline conversation changes if cutting 60% of scope saves 20% of time). Target ~80% of the slot; a cut that exactly fills the timeline has pre-spent the buffer reality always invoices.

6. **Communicate the cut as a decision, not an apology:** stakeholders hear WHAT ships, for whom, when; deferred with triggers; dropped with reasons (`stakeholder-update` carries it). The sales-facing version needs explicit don't-promise-the-deferred markings — a cut communicated vaguely gets un-cut by promise.

7. **After ship, judge the cuts with the metric:** did the kernel deliver? Which deferred triggers actually fired? (Usually fewer than half — the quiet vindication of cutting.) Which cut generated real pain? (Restore with evidence, ahead of the speculative rest.) This loop is what makes the NEXT scoping conversation fast and trusted.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Keep all the features — just skip some tests and review" | That's cutting quality, the invisible cut that ships the whole surface area rotten. Visible scope shrinks; the kernel stays solid. The deadline made honestly is smaller, not sloppier. |
| "Cut the instrumentation for now — it's not user-facing" | Then the feature can't prove itself, the deferred list can't re-prioritize with data, and the retro is vibes. The metric events are part of the kernel by definition (step 2's do-not-cut list). |
| "Everything left is essential — it's all kernel" | If the success metric moves without it, it's an amplifier, whoever loves it. The kernel test is mechanical precisely so pet features can't self-classify. |
| "Just add this one thing back — it's small" | Five small restorations silently rebuild the original scope by week 4. The disposition list is the contract; additions reopen it explicitly or the cut was theater. |
| "Cut the expensive parts — biggest schedule win" | Sometimes the expensive part IS the kernel, and shipping the cheap husk 'on time' delivers nothing. When kernel-cost exceeds the slot, the honest move is moving the date. |
| "MVP means minimum — strip it to the bone" | Minimum without viable completes nobody's task and validates only that broken things don't retain users. The coherence walk (step 4) is the viability check. |

## Red Flags

- The "cut" plan touching test coverage, error handling, or security instead of features.
- A phase-2 list with no triggers, no tickets, no reasons.
- Scope quietly regrowing through week-by-week additions.
- A reduced scope that fills 100% of the remaining timeline.
- The user's workflow walk hitting a dead middle ("upload works, errors just... fail").
- No success-metric events in the shipped v1.

## Verification

- [ ] Kernel sentence agreed and written — derived from the success metric.
- [ ] Every cut classified on an axis with a disposition (deferred+trigger or dropped+reason) — the list published.
- [ ] Do-not-cut list intact: data/money error paths, security, instrumentation — checked in the shipped diff.
- [ ] Coherence walk done: the primary path completes end-to-end — walkthrough noted.
- [ ] Re-estimate lands ≤80% of the slot — numbers shown.
- [ ] Post-ship cut-review completed: triggers checked, restorations evidence-based — retro linked.

## Example

Feature: payout reconciliation (from the `prd-writing` example), specced at 9 weeks against a 6-week slot. Kernel sentence: "ops manager reconciles one day's payouts without a spreadsheet." Cuts: audience → US-entity accounts only (EU tax formats deferred, trigger: "3+ EU enterprise requests" — the estimate's biggest line); variety → CSV export only (in-app view deferred); automation → refund-adjustment edge case via support macro for launch month (concierge — est. 40 cases/month); the settings page dropped outright. NOT cut: per-row error reporting (data-integrity path) and the reconciliation-time instrumentation. Re-estimate: 5.5 weeks — the margin absorbed a real one-week auth surprise. Post-ship: success metric hit at week 7; of four deferred items, one trigger fired (EU requests — built next quarter with evidence in hand), the in-app view's never did (support macro logs showed 11 cases/month, not 40 — deferred again, cheaply, with data). The step-7 retro took twenty minutes because every cut had a written disposition to check.

## Related skills

- `prd-writing` — the kernel's source of truth.
- `user-story-slicing` — sequencing what survives the cut.
- `roadmap-communication` — carrying the deferral news outward.
- `prioritization-frameworks` — where the deferred list re-competes.
