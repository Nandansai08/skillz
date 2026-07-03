---
name: citation-management
description: >
  Use when setting up or fixing how sources are tracked — keeping every
  claim traceable to its source, reference tooling habits, link rot
  defense, so documents survive scrutiny. Triggers: "manage my references",
  "citation workflow", "where did this claim come from", "set up Zotero",
  "the links in our doc are dead", "make this doc auditable".
---

# Citation Management

## When to use this skill
- Starting research/writing that will face scrutiny (papers, decision memos, public posts, compliance docs).
- Retro-fixing a document full of unsourced claims and dead links.
- NOT judging source quality (`source-credibility-check`) or synthesizing them (`note-synthesis`) — this is the plumbing that keeps claim→source connections intact through drafts, years, and reviews.

## Prerequisites
- Honesty about your workflow's weight class: a solo decision memo and a dissertation need different machinery — over-tooling kills the habit that matters more than any tool.

## Workflow

1. **Capture at encounter, with retrieval fields, or lose it:** the iron rule — the moment a source enters your thinking, it enters the system (browser extension to Zotero/reference manager for the heavy class; a running `sources.md` with a one-line entry for the light class). Minimum viable record: locator (URL/DOI/ISBN), accessed date, author/venue, and the one-line "why kept" — the ten-second entry that prevents the two-hour "where did I see that benchmark?" archaeology, which is the tax this whole skill eliminates. For web sources, capture the *specific* page, not the site.

2. **Bind claims to sources at writing time, not cleanup time:** every factual claim in a draft carries its pointer AS IT'S WRITTEN (inline key, footnote, or `[src: sources.md#L14]` in the light class — the format matters less than the timing). "I'll add citations later" is the workflow's classic lie: later, you'll remember the claim and not the source, and the cleanup pass silently converts sourced-claims into vibes-you-once-had-a-source-for. The `note-synthesis` claim file (its step 1 pointers) makes this nearly free when that skill ran upstream.

3. **Cite at the resolution the claim needs:** page/section/timestamp for specific facts ("§4.2", "at 14:30 in the recording", "Table 3") — a 400-page-PDF citation for one number is technically sourced and practically unverifiable; whole-work citations only for whole-work claims ("the book argues..."). The resolution rule is what makes review efficient: your future auditor (often you) should reach the exact evidence in one hop.

4. **Defend against link rot — the web citation's guaranteed decay:** archive at capture (Wayback Machine save, or local PDF snapshot for anything load-bearing — the vendor pricing page and the blog post WILL change or vanish, usually before the doc's next review); prefer stable locators where they exist (DOI > publisher URL > blog URL); record the accessed date always (the claim was true of the page THEN — the date scopes your liability when the page mutates). For internal sources: permalinks over channel-scroll links, and the doc/version, not "the spreadsheet."

5. **Distinguish what KIND of support each citation claims:** the source states this directly / the source's data implies this (my inference — `note-synthesis` step 2's marks surviving into the document) / the source is one example of this pattern / see-also background. Uniform citation formatting hides these differences, and reviewers' worst discoveries are direct-citation formatting on inference-grade support ("the paper doesn't say that"). A word of framing in the prose ("X reports...", "consistent with...", "e.g.") carries the distinction at zero cost.

6. **Run the pre-ship audit — the pass that catches the drift:** every claim-shaped sentence checked for a pointer (unsourced ones get sourced, hedged, or cut — `source-credibility-check`'s verdicts applied); every pointer spot-checked that it still says what you cite it for (drafts evolve; the paragraph got rewritten and the citation now supports a neighboring claim, not this one — citation drift is invisible without the check); links resolve or archives substituted. On anything high-stakes, have someone else run five random claim→source traces — their hit rate is the document's audit-worthiness measured.

7. **Maintain the library as an asset, lightly:** one system (two half-used managers = zero systems), dedupe on entry, tag by project/topic at capture (retroactive taxonomy never happens), and prune-by-archive rather than delete. The payoff compounds exactly like `note-synthesis` step 7's claim-base: the third project on adjacent ground starts with a populated, trusted library — and the recurring claim that resurfaces yearly (`source-credibility-check`'s example) costs one lookup instead of one investigation each time.

## Common pitfalls
- Capture-later syndrome: fifteen tabs of "I'll save the good ones tonight" — the encounter-time rule (step 1) exists because tonight's memory of which-tab-said-what is already corrupted.
- The bibliography that outgrew its claims: 60 impressive references, but the DOCUMENT's sentences don't point INTO them — a reading list stapled to an essay. Binding (step 2) is per-claim, not per-document.
- Unverifiable resolution: "(Smith 2024)" for one number in a book — sourced in theory, uncheckable in practice (step 3).
- Naked URLs as citations for load-bearing claims: the pricing page changed, the post was deleted, the startup died — and the doc's key number now cites a 404 (step 4's archive habit is cheap; the alternative is retraction).
- Inference in citation costume: your reasonable extrapolation formatted identically to the source's direct statement — the reviewer who catches one downgrades every citation in the document (step 5).
- Tool maximalism as procrastination: three days configuring the reference manager's tag hierarchy, zero sources captured — the habit (capture at encounter, bind at writing) IS the system; tooling serves it or it's décor.

## Example
Context: a build-vs-buy memo headed to exec review, drafted from three weeks of research — historically the team's memos died in review on "source?" challenges. Setup (light class, deliberately): one `sources.md`, 31 entries captured at encounter with accessed-dates and why-kept lines; claims bound inline as `[s7]`-style keys during drafting, not after. The step-6 pre-ship audit earned its slot three ways: caught two unsourced claim-sentences (one got sourced — a five-minute hunt while memory was warm; one got cut as unfindable — better cut by the author than caught by the CFO); caught one citation-drift case (a rewritten paragraph citing the vendor's SLA for a claim about *support response times* — the neighboring claim's source); and the link check archived four pages including the vendor's pricing page — which, three weeks later at contract negotiation, HAD changed, and the dated snapshot of the old pricing became a negotiating document in its own right. Review outcome: every challenge answered in one hop, the memo's recommendation adopted, and the sources file seeded the renewal analysis two quarters on. Total plumbing overhead across three weeks: under two hours.

## Related skills
- `source-credibility-check` — judging what the plumbing carries.
- `note-synthesis` — the claim layer these pointers attach to.
- `literature-review` — the many-source process needing this most.
- `architecture-decision-record` — decision docs whose links deserve the same rot defense.
