---
name: source-credibility-check
description: >
  Use when deciding how much to trust a specific source or claim — primary
  vs secondary, incentive analysis, recency, claim-chasing to origin.
  Triggers: "is this source reliable", "can I cite this", "fact-check this
  claim", "this stat seems off", "who's behind this study",
  "verify before we repeat it".
---

# Source Credibility Check

## When to use this skill
- A specific claim/source is about to enter your decision, document, or public statement.
- A statistic "everyone knows" needs its provenance checked before you repeat it.
- NOT for surveying a whole field (`literature-review` — which invokes this per-source) — this is the single-source audit, minutes not days.

## Prerequisites
- The claim stated precisely (numbers, scope, conditions) — vague claims can't be checked, only vibed about.

## Workflow

1. **Chase the claim to its primary source — the non-negotiable first move:** follow the citation chain until you reach the origin (the study, the dataset, the filing, the person who measured). Each hop in the chain is a mutation opportunity — the blog cites the article cites the press release cites the abstract, and "27% under some conditions in mice" arrives as "cures cancer." If the chain dead-ends (no citation, "studies show", a 404), the claim is currently unsourced — treat it as such regardless of how many outlets repeat it (repetition is not corroboration when everyone copied the same origin).

2. **Read what the primary source actually says vs what's claimed of it:** scope (population, conditions, timeframe — the finding about US enterprises in 2019 quoted as a universal law), effect size vs mere existence ("statistically significant" 0.3% differences marketed as breakthroughs), and the original's OWN hedges and limitations section (authors usually confess; quoters usually don't). The most common credibility failure isn't fake sources — it's real sources overclaimed.

3. **Run the incentive audit on the origin:** who paid (funding statements, the vendor behind the "independent" benchmark, the industry group behind the institute with the neutral name); what the source gains if you believe it (selling something? litigating something? building a career on the theory?); and the softer versions — the researcher's entire lab premised on the hypothesis, the outlet's traffic model rewarding alarm. Incentive doesn't falsify (funded research can be sound) — it tells you where to CHECK harder: the methodology choices that happen to favor the funder are where sponsored work bends.

4. **Grade the evidence type the source represents:** measurement > testimony; systematic > anecdotal; pre-registered > post-hoc; replicated > single-shot; primary data > reconstruction. Practitioner equivalents: reproducible benchmark with published methodology > experience report > demo > vendor whitepaper. And check the denominator discipline (`metric-definition`'s habits): survivorship framing ("90% of our successful customers..."), base rates omitted, absolute vs relative risk conflated ("doubles the risk" of something with a 0.001% base).

5. **Check recency against the claim's decay rate:** facts age at different speeds — anatomy is stable, JavaScript-framework benchmarks rot in a year, security guidance in months. The question isn't "is this old?" but "has the ground moved since?" — a 2019 claim about tooling costs is archaeological; check whether the source's version/conditions still describe the present (and whether newer work has replicated, refined, or reversed it — the forward citation search from `literature-review` step 2).

6. **Corroborate INDEPENDENTLY — verify the sources don't share an origin:** two genuinely independent measurements agreeing is strong; forty articles all tracing to one press release is one source wearing forty hats (the citation-chase from step 1, run on the corroborators). Disagreement between independent sources isn't failure — it bounds the truth and often reveals the moderating variable (the same payoff as `literature-review` step 5's conflicts).

7. **Assign the trust verdict and record the trail:** solid (use it) / usable-with-caveats (state them inline where you cite it) / unverified (don't repeat as fact; "one vendor study claims..." if needed at all) / debunked. One line of provenance saved with the claim ("traced to: primary at [link], scope note: US-only, funder: X") — because you or a colleague WILL meet this claim again, and the five minutes of chase shouldn't be re-spent (`citation-management` holds the trail).

## Common pitfalls
- Outlet-prestige proxy: trusting the claim because a famous publication carried it — outlets syndicate press releases too; the chain-chase (step 1) applies to everyone.
- Citation-count-as-truth: heavily-cited papers get cited for existing, including famously-failed-to-replicate ones; check whether the citations are supportive, critical, or reflexive.
- The incentive pass applied only to disliked sources — your side's think tank has a funding statement too; asymmetric skepticism is confirmation bias with a methodology.
- Statistical costume: precise-sounding numbers ("73.4% of engineers...") from an n=87 self-selected poll — precision implies rigor to readers; check the n and the frame before the decimal impresses you (`survey-design` step 5's honesty, demanded from others).
- Recency snobbery in reverse: dismissing the foundational old source for a shallow new one — decay-rate thinking (step 5) cuts both directions.
- Checking only what you'll cite, not what you already believe: the stat in your own deck for two years, origin unknown, repeated at board level. Audit inward occasionally.

## Example
Claim inbound for the exec deck: "poor code quality costs companies $85B annually" — cited from a respected tech site. Chase: article → older article → a vendor's "research report" → the trail's origin: a decade-old estimate extrapolating a small consultancy survey, methodology unavailable, produced by a company selling code-quality tooling (incentive audit: existential). Scope check: the number covered a different construct entirely ("developer time on ALL maintenance" — including normal feature evolution, not "poor quality"). Independent corroboration: none — every repetition traced to the same origin (forty hats, one head). Verdict: unverified-bordering-debunked; removed from the deck. Replacement: the team's OWN measurable ("we spend ~30% of sprint capacity on defect rework — internal data, last two quarters"), which was smaller, honest, and — being checkable — survived the CFO's questions where the $85B would have died. Provenance line archived; the claim has since resurfaced twice in other decks and cost five seconds each time.

## Related skills
- `literature-review` — the many-source process this audits inside.
- `citation-management` — where the provenance trails live.
- `metric-definition` — the denominator instincts for step 4.
- `natural-prose-editing` — killing the vague-attribution phrasing that smuggles weak sources.
