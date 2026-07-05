---
name: source-credibility-check
description: >
  Use when deciding how much to trust a specific source or claim —
  primary vs secondary, incentive analysis, recency, claim-chasing to
  origin. Triggers: "is this source reliable", "can I cite this",
  "fact-check this claim", "this stat seems off", "who's behind this
  study", "verify before we repeat it". NOT for surveying a whole field
  (see literature-review, which invokes this per-source) — this is the
  single-source audit, minutes not days.
---

# Source Credibility Check

## Overview

Chase the claim to its origin — every hop in a citation chain is a mutation opportunity, and forty outlets repeating one press release is one source wearing forty hats. The commonest failure isn't fake sources; it's real sources overclaimed, caught by reading what the primary actually says versus what's quoted of it.

## When to Use

- A specific claim/source is about to enter your decision, document, or public statement.
- A statistic "everyone knows" needs provenance before you repeat it.

**When NOT to use:**
- Whole-field surveys — `literature-review`.

## Prerequisites

- The claim stated precisely (numbers, scope, conditions) — vague claims can't be checked, only vibed about.

## The Workflow

1. **Chase the claim to its primary source — the non-negotiable first move.** Follow the chain until you reach the origin (the study, the dataset, the filing). Each hop mutates: the blog cites the article cites the press release cites the abstract, and "27% under some conditions in mice" arrives as "cures cancer." Chain dead-ends (no citation, "studies show", a 404) = currently unsourced — treat as such regardless of repetition count; repetition is not corroboration when everyone copied one origin.

2. **Read what the primary actually says vs what's claimed of it:** scope (population, conditions, timeframe — the 2019-US-enterprise finding quoted as universal law), effect size vs mere existence, and the original's OWN limitations section (authors confess; quoters don't). The most common credibility failure is real sources overclaimed.

3. **Run the incentive audit on the origin:** who paid (the vendor behind the "independent" benchmark, the industry group behind the neutrally-named institute); what the source gains if believed; the softer versions (the lab premised on the hypothesis, the outlet's traffic model). Incentive doesn't falsify — it tells you where to CHECK harder: the methodology choices that happen to favor the funder are where sponsored work bends. And apply it symmetrically — your side's think tank has a funding statement too.

4. **Grade the evidence type:** measurement > testimony; systematic > anecdotal; pre-registered > post-hoc; replicated > single-shot. Practitioner: reproducible benchmark with published methodology > experience report > demo > whitepaper. Check denominator discipline (`metric-definition`'s habits): survivorship framing, missing base rates, relative-risk theater ("doubles the risk" of a 0.001% base). Precision costume too: "73.4% of engineers" from an n=87 self-selected poll — check the n before the decimal impresses you.

5. **Check recency against the claim's decay rate:** facts age at different speeds — anatomy is stable, framework benchmarks rot in a year, security guidance in months. The question isn't "is this old?" but "has the ground moved?" — and it cuts both ways: dismissing the foundational old source for a shallow new one is recency snobbery.

6. **Corroborate INDEPENDENTLY — verify the sources don't share an origin:** two genuinely independent measurements agreeing is strong; forty articles tracing to one press release is one source in forty hats (run the chain-chase on the corroborators). Disagreement between independents isn't failure — it bounds the truth and often reveals the moderating variable.

7. **Assign the trust verdict and record the trail:** solid / usable-with-caveats (stated inline where cited) / unverified ("one vendor study claims...", if used at all) / debunked. One provenance line saved with the claim ("traced to: [link]; scope: US-only; funder: X") — you'll meet this claim again, and the five-minute chase shouldn't be re-spent (`citation-management` holds the trail).

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "It's in [famous outlet] — that's vetting enough" | Famous outlets syndicate press releases too. The chain-chase applies to everyone; prestige is a prior, not a verification. |
| "It's cited everywhere — must be established" | The example's $85B stat was cited everywhere and traced to one vendor's methodology-free decade-old extrapolation. Repetition without independence is one source amplified. |
| "The study exists, so the claim stands" | The study says 27%-in-mice-under-conditions; the claim says cures-cancer. Existence-of-source and support-of-claim are different checks, and step 2 is the second one. |
| "Checking funders is ad hominem — judge the work" | Incentive doesn't falsify; it aims the scrutiny at the methodology choices that happen to favor the funder — which is exactly where sponsored work bends. |
| "No time to verify — everyone uses this number" | Five minutes of chase versus repeating a debunkable number at board level. The example's replacement stat survived the CFO precisely because it was checkable. |
| "I only need to check the sources I disagree with" | Asymmetric skepticism is confirmation bias with a methodology. The stat in your own deck for two years deserves the same chase. |

## Red Flags

- Claims entering documents with "studies show" and no link.
- Citation chains nobody has walked to the end.
- Precise decimals from tiny self-selected samples.
- "Independent" reports with vendor logos on the last page.
- The same stat appearing everywhere with identical phrasing (one origin, many hats).
- Your own deck's flagship number, provenance unknown.

## Verification

- [ ] Chain walked to the primary; origin linked (or the claim marked unsourced).
- [ ] Primary's scope/limitations read; claim-vs-source delta noted.
- [ ] Funder/incentive identified; scrutiny aimed accordingly.
- [ ] Evidence type graded; denominator/base-rate checks done.
- [ ] Independence of corroborators verified (chains don't converge).
- [ ] Verdict assigned; provenance line archived with the claim.

## Example

Claim inbound for the exec deck: "poor code quality costs companies $85B annually" — cited from a respected tech site. Chase: article → older article → a vendor's "research report" → the origin: a decade-old estimate extrapolating a small consultancy survey, methodology unavailable, produced by a company selling code-quality tooling (incentive audit: existential). Scope check: the number covered a different construct entirely ("developer time on ALL maintenance" — including normal feature evolution). Independent corroboration: none — every repetition traced to the same origin. Verdict: unverified-bordering-debunked; removed from the deck. Replacement: the team's OWN measurable ("~30% of sprint capacity on defect rework — internal data, two quarters"), which was smaller, honest, and — being checkable — survived the CFO's questions where the $85B would have died. Provenance line archived; the claim has since resurfaced twice in other decks and cost five seconds each time.

## Related skills

- `literature-review` — the many-source process this audits inside.
- `citation-management` — where the provenance trails live.
- `metric-definition` — the denominator instincts for step 4.
- `natural-prose-editing` — killing the vague-attribution phrasing that smuggles weak sources.
