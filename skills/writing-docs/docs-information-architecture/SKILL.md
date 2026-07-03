---
name: docs-information-architecture
description: >
  Use when organizing a documentation set — separating tutorials, how-to
  guides, reference, and explanation (Diátaxis), deciding what goes where,
  restructuring a docs site users can't navigate. Triggers: "organize our
  docs", "docs site structure", "tutorial vs guide vs reference", "users
  can't find anything in the docs", "documentation strategy".
---

# Docs Information Architecture

## When to use this skill
- A documentation set has outgrown its structure (or never had one) and users can't find answers.
- Deciding where a new piece of documentation belongs.
- NOT for writing the individual documents — `readme-authoring`, `api-documentation`, `runbook-authoring` cover those; this skill is the map they live on.

## Prerequisites
- Evidence of how the docs currently fail: support questions that docs should have answered, search queries on the docs site, "where do I find X" messages.

## Workflow

1. **Sort all content against the Diátaxis compass — two questions, four quadrants:** is the reader *learning or working*? Does the doc convey *steps or understanding*?
   - **Tutorial** (learning + steps): a guaranteed-success guided lesson — "build your first integration in 20 minutes." Author controls everything; reader follows exactly.
   - **How-to guide** (working + steps): task recipe for someone competent — "rotate credentials", "migrate from v1". Assumes context, states its assumptions, gets to the point.
   - **Reference** (working + understanding): complete, accurate description — API params, config options, CLI flags. Optimized for lookup, not reading; generated where possible (`api-documentation` step 5).
   - **Explanation** (learning + understanding): why it works this way — architecture, trade-offs, concepts. The ADR log's public face (`architecture-decision-record`).
   The framework's real value is diagnostic: most bad docs are two quadrants fused — the "tutorial" that digresses into every configuration option, the reference full of narrative.

2. **Audit the existing corpus with a three-label pass:** for each page — which quadrant(s) does it contain, is it findable from where users start, is it current? Output: a spreadsheet with keep / split (multi-quadrant pages) / merge (five stale half-pages about auth) / delete (wrong docs are worse than missing ones — `onboarding-doc-design` step 6's rule). The audit usually finds the corpus is 70% reference-flavored, missing how-tos almost entirely — because engineers write down what things ARE, and users arrive asking how to DO.

3. **Fill by user-journey priority, not quadrant symmetry:** rank real tasks by frequency × frustration (the support-ticket and search-query evidence from prerequisites is the ranking data). Typical priority for developer products: getting-started tutorial (one, excellent) → the top-10 how-tos → reference completeness → explanations. You don't need four beautiful quadrants; you need the pages your users' actual journeys hit.

4. **Design navigation for the working reader (the majority):** they arrive by search with a task in mind. Consequences: titles are task-phrased in the user's vocabulary ("Rotate API credentials", never "Credential lifecycle management"); each page states its assumptions up top and links prerequisites rather than inlining them; sidebar organized by user goal, not by your team's org chart or the codebase's module structure — the internal-structure sidebar is the most common architecture failure and it's invisible to insiders.

5. **Cross-link at the quadrant seams, by convention:** tutorial ends with "where to go next" (the how-tos); how-tos link the reference for every parameter they touch; reference links the explanation for every "why is it like this?" surprise. The links are what let each page stay pure (step 1) without stranding readers.

6. **Establish the placement rule for new content** — one question in the PR template: "which quadrant, and which existing page did you check first?" Plus ownership per section and the freshness discipline: last-verified dates on how-tos (they rot fastest — they encode the UI/CLI of a specific version), CI-tested code samples where feasible (`api-documentation` step 6).

7. **Measure the architecture like a product:** docs-site search queries with zero results (missing pages, named by users), support tickets that a doc should have caught (each is a how-to backlog item — `api-documentation` step 7's loop), and time-to-first-success for the tutorial (watch one new user run it, quarterly). Restructure when the evidence says so, not when the structure feels stale to its authors.

## Common pitfalls
- The fused page: a "getting started" that's 20% tutorial, 50% reference dump, 30% architecture essay — serves all four readers badly. Split along quadrant lines.
- Writing more tutorials because they're satisfying to write, when users are stuck on day-30 tasks needing how-tos. One great tutorial; then how-tos until the ticket evidence quiets.
- Sidebar mirroring the codebase (`/docs/services/auth-svc/`) — perfectly logical to the team, meaningless to a user whose task is "let users log in."
- Deleting nothing: eight years of pages, three auth guides (two wrong), and every search returns all three. Curation IS the architecture; the delete column of step 2's audit is where trust is rebuilt.
- Assuming search solves structure ("they'll just search") — search finds pages, but only structure makes the *right* page recognizably right when three candidates appear.
- Big-bang restructures every two years instead of step 6's placement discipline — the entropy that forced the restructure resumes the day after it ships.

## Example
Developer-platform docs: 210 pages grown over five years, support running ~25 docs-answerable tickets/week, top site search with zero results: "webhook retry". Audit: 140 reference-ish pages (largely current), 4 tutorials (three broken — deprecated SDK), 11 how-tos, dozens of fused pages. Actions: one tutorial rebuilt and CI-tested, two deleted; ticket analysis yielded a top-12 how-to list ("verify webhook signatures", "handle rate limits", "migrate v1→v2" — none existed); 30 fused pages split with cross-links per step 5; sidebar rebuilt around five user goals (Authenticate / Accept payments / Handle events / Test / Go live) replacing the six-team org-chart structure; 41 pages deleted. Eight weeks of evidence after: docs-answerable tickets ~25 → ~9/week, zero-result searches down 70%, and the "webhook retry" how-to — written from the ticket transcripts — became the site's #3 page. The placement question went into the docs PR template; quadrant drift since: two pages, caught in review.

## Related skills
- `api-documentation` — the reference quadrant's deep practice.
- `readme-authoring` — the front door that routes into this architecture.
- `onboarding-doc-design` — the same audience-journey thinking, internal edition.
- `architecture-decision-record` — raw material for the explanation quadrant.
