---
name: citation-management
description: >
  Use when setting up or fixing how sources are tracked — keeping every
  claim traceable to its source, reference tooling habits, link rot
  defense, so documents survive scrutiny. Triggers: "manage my
  references", "citation workflow", "where did this claim come from",
  "set up Zotero", "the links in our doc are dead", "make this doc
  auditable". NOT judging source quality (see source-credibility-check)
  or synthesizing sources (see note-synthesis) — this is the plumbing.
---

# Citation Management

## Overview

Two habits carry the whole discipline: capture at encounter (tonight's memory of which-tab-said-what is already corrupted) and bind claims to sources at writing time ("I'll add citations later" is the workflow's classic lie). Everything else — resolution, rot defense, the audit — protects what those two habits create.

## When to Use

- Starting research/writing that will face scrutiny (papers, decision memos, public posts, compliance docs).
- Retro-fixing a document full of unsourced claims and dead links.

**When NOT to use:**
- Judging quality — `source-credibility-check`.
- Synthesis — `note-synthesis`.

## Prerequisites

- Honesty about weight class: a solo memo and a dissertation need different machinery — over-tooling kills the habit that matters more than any tool.

## The Workflow

1. **Capture at encounter, with retrieval fields, or lose it:** the moment a source enters your thinking, it enters the system (browser extension to a reference manager, or a running `sources.md` for the light class). Minimum record: locator (URL/DOI), accessed date, author/venue, one-line "why kept." The ten-second entry prevents the two-hour "where did I see that benchmark?" archaeology — the tax this whole skill deletes.

2. **Bind claims to sources at writing time, not cleanup time:** every factual claim carries its pointer AS IT'S WRITTEN (inline key, footnote, `[s7]`-style). "Later" means you'll remember the claim and not the source, and cleanup silently converts sourced-claims into vibes-you-once-had-a-source-for. The `note-synthesis` claim file makes this nearly free when it ran upstream.

3. **Cite at the resolution the claim needs:** page/section/timestamp for specific facts ("§4.2", "Table 3") — a 400-page-PDF citation for one number is technically sourced and practically unverifiable; whole-work citations only for whole-work claims. The auditor (often future you) reaches the exact evidence in one hop.

4. **Defend against link rot — the web citation's guaranteed decay:** archive at capture (Wayback save or local snapshot for anything load-bearing — the pricing page and the blog post WILL change or vanish); stable locators where they exist (DOI > publisher URL > blog); accessed dates always (the claim was true of the page THEN). Internal sources: permalinks and doc versions, not "the spreadsheet."

5. **Distinguish what KIND of support each citation claims:** states-directly / implies-via-data (my inference — `note-synthesis` step 2's marks surviving into the document) / one-example-of / see-also. Uniform citation formatting hides these; reviewers' worst discoveries are direct-citation formatting on inference-grade support ("the paper doesn't say that"). A word of framing ("X reports...", "consistent with...", "e.g.") carries the distinction free.

6. **Run the pre-ship audit — the pass that catches the drift:** every claim-shaped sentence checked for a pointer (unsourced → sourced, hedged, or cut); every pointer spot-checked that it still says what you cite it for (drafts evolve; the rewritten paragraph's citation now supports a neighboring claim — citation drift is invisible without the check); links resolve or archives substituted. High-stakes: someone else runs five random claim→source traces — their hit rate is the document's audit-worthiness, measured.

7. **Maintain the library as an asset, lightly:** one system (two half-used managers = zero), dedupe on entry, tag at capture (retroactive taxonomy never happens), prune-by-archive. The payoff compounds: the third project on adjacent ground starts populated, and the recurring claim costs one lookup instead of one investigation each time it resurfaces.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "I'll save the good tabs tonight" | Tonight's memory of which-tab-said-what is already corrupted, and three tabs are closed. Capture-at-encounter is ten seconds; reconstruction is the two-hour tax. |
| "Citations are a cleanup pass before shipping" | At cleanup you remember the claim, not the source — and the pass quietly converts sourced-claims into confident vibes. Binding happens at writing or the trail never existed. |
| "(Smith 2024) covers it — the number's in there somewhere" | A 400-page citation for one figure is unverifiable in practice, which reviewers treat as unverified. Page-level resolution is what makes the audit one hop. |
| "The URL works today — that's what matters" | The pricing page mutates and the startup's blog dies, always before the doc's next review. The example's archived snapshot became a negotiating document; the naked URL would have become a 404. |
| "My extrapolation is obviously reasonable — cite the source" | Inference in direct-citation costume is the reviewer's worst discovery, and one instance discounts every citation in the document. 'Consistent with...' costs two words and keeps the trust. |
| "A better tool will fix my reference chaos" | Three days of tag-hierarchy configuration, zero sources captured. The habits (capture at encounter, bind at writing) ARE the system; tooling serves them or it's décor. |

## Red Flags

- Fifteen open tabs "to save later."
- A bibliography longer than the claims that point into it.
- Claim-shaped sentences with no pointers, surviving review rounds.
- Naked URLs on load-bearing claims, no archives, no dates.
- Inferences formatted identically to direct quotes.
- "Where did this number come from?" asked about your own document.

## Verification

- [ ] Every source captured at encounter with the four fields — spot-check the last five additions' dates vs first-use dates.
- [ ] Every factual claim carries an inline pointer — a full pass confirms zero orphans.
- [ ] Specific facts cited at page/section/timestamp resolution — spot-check three.
- [ ] Load-bearing web sources archived; accessed dates present — checked.
- [ ] Support-kind framing present where citations back inference — reviewed.
- [ ] Pre-ship audit run (drift spot-checks, link resolution); high-stakes docs: independent five-trace test passed — results noted.

## Example

Context: a build-vs-buy memo headed to exec review, drafted from three weeks of research — historically the team's memos died in review on "source?" challenges. Setup (light class, deliberately): one `sources.md`, 31 entries captured at encounter with accessed-dates and why-kept lines; claims bound inline as `[s7]`-style keys during drafting. The step-6 audit earned its slot three ways: caught two unsourced claim-sentences (one sourced in five minutes while memory was warm; one cut as unfindable — better cut by the author than caught by the CFO); caught one citation-drift case (a rewritten paragraph citing the vendor's SLA for a claim about *support response times*); and the link check archived four pages including the vendor's pricing page — which, three weeks later at contract negotiation, HAD changed, and the dated snapshot became a negotiating document in its own right. Review outcome: every challenge answered in one hop; the sources file seeded the renewal analysis two quarters on. Total plumbing overhead: under two hours.

## Related skills

- `source-credibility-check` — judging what the plumbing carries.
- `note-synthesis` — the claim layer these pointers attach to.
- `literature-review` — the many-source process needing this most.
- `architecture-decision-record` — decision docs whose links deserve the same rot defense.
