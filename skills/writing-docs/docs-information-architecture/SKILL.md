---
name: docs-information-architecture
description: >
  Use when organizing a documentation set — separating tutorials, how-to
  guides, reference, and explanation (Diátaxis), deciding what goes
  where, restructuring a docs site users can't navigate. Triggers:
  "organize our docs", "docs site structure", "tutorial vs guide vs
  reference", "users can't find anything in the docs", "documentation
  strategy". NOT for writing the individual documents — readme-authoring,
  api-documentation, runbook-authoring cover those; this is the map.
---

# Docs Information Architecture

## Overview

Most bad docs are two Diátaxis quadrants fused — the tutorial that digresses into every config option, the reference full of narrative — and most bad docs sites are organized by the team's org chart instead of the user's task. The architecture is quadrant separation plus journey-priority filling plus ruthless deletion.

## When to Use

- A documentation set has outgrown its structure (or never had one) and users can't find answers.
- Deciding where a new piece of documentation belongs.

**When NOT to use:**
- Writing the individual documents — the per-genre skills.

## Prerequisites

- Evidence of how the docs currently fail: support questions docs should have answered, docs-site search queries, "where do I find X" messages.

## The Workflow

1. **Sort all content against the Diátaxis compass — two questions, four quadrants:** is the reader *learning or working*? Does the doc convey *steps or understanding*?
   - **Tutorial** (learning + steps): a guaranteed-success guided lesson. Author controls everything.
   - **How-to guide** (working + steps): task recipe for someone competent — states its assumptions, gets to the point.
   - **Reference** (working + understanding): complete, accurate description — optimized for lookup, generated where possible.
   - **Explanation** (learning + understanding): why it works this way — architecture, trade-offs (the ADR log's public face).
   The framework's real value is diagnostic: most bad docs are two quadrants fused.

2. **Audit the existing corpus with a three-label pass:** per page — which quadrant(s), findable from where users start?, current? Output: keep / split (multi-quadrant pages) / merge (five stale half-pages about auth) / delete (wrong docs are worse than missing ones). The usual finding: 70% reference-flavored, how-tos nearly absent — engineers write what things ARE; users arrive asking how to DO.

3. **Fill by user-journey priority, not quadrant symmetry:** rank real tasks by frequency × frustration (the ticket and search evidence is the ranking data). Typical priority: one excellent getting-started tutorial → the top-10 how-tos → reference completeness → explanations. You need the pages users' journeys hit, not four beautiful quadrants.

4. **Design navigation for the working reader (the majority):** they arrive by search with a task. Consequences: task-phrased titles in the user's vocabulary ("Rotate API credentials", never "Credential lifecycle management"); assumptions stated up top with prerequisites linked; sidebar organized by user goal — not by your org chart or the codebase's module structure (the most common failure, invisible to insiders).

5. **Cross-link at the quadrant seams, by convention:** tutorial ends with "where next" (the how-tos); how-tos link the reference for every parameter touched; reference links the explanation for every "why is it like this?" surprise. The links let each page stay pure without stranding readers.

6. **Establish the placement rule for new content:** one question in the docs PR template — "which quadrant, and which existing page did you check first?" Plus ownership per section, last-verified dates on how-tos (they rot fastest — they encode a specific version's UI), CI-tested code samples where feasible.

7. **Measure the architecture like a product:** zero-result search queries (missing pages, named by users), support tickets a doc should have caught (each a how-to backlog item), time-to-first-success on the tutorial (watch one new user quarterly). Restructure when evidence says so, not when the structure feels stale to its authors.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "One comprehensive getting-started page covers everything" | The fused page — 20% tutorial, 50% reference dump, 30% architecture essay — serves all four readers badly. Splitting along quadrant lines is the fix, not more content on the pile. |
| "Organize by our services — it's the logical structure" | Logical to the org chart, meaningless to a user whose task is 'let users log in.' Goal-based navigation is the architecture decision that moves the metrics most. |
| "Users will just search" | Search finds pages; only structure makes the RIGHT page recognizably right when three candidates appear — including the two stale ones you didn't delete. |
| "Never delete docs — someone might need that page" | Eight years of pages means three auth guides, two wrong, all ranking. Curation IS the architecture; the delete column is where user trust gets rebuilt. |
| "Writing tutorials is the best use of doc time" | Tutorials are satisfying to write while users are stuck on day-30 tasks needing how-tos. The ticket evidence allocates the effort; author enjoyment doesn't. |
| "We'll restructure properly next year" | Big-bang restructures decay from day one without the placement rule (step 6). The PR-template question is what makes the structure self-maintaining. |

## Red Flags

- Docs-answerable support tickets recurring weekly.
- Top zero-result search terms untracked or unowned.
- Sidebar mirroring the repo's directory tree.
- Pages that are tutorial-reference-explanation smoothies.
- Three documents on the same topic from different eras, all live.
- New docs merged with no quadrant/placement question asked.

## Verification

- [ ] Corpus audit table exists: quadrant, findability, currency, disposition per page — linked.
- [ ] Deletions executed, not deferred — count noted.
- [ ] Journey-priority backlog built from ticket/search evidence — the top-10 how-to list linked.
- [ ] Sidebar reorganized by user goal — before/after structure shown.
- [ ] Placement question live in the docs PR template — link.
- [ ] Baseline metrics captured (zero-result rate, docs-answerable tickets/week) for the next review.

## Example

Developer-platform docs: 210 pages grown over five years, support running ~25 docs-answerable tickets/week, top zero-result search: "webhook retry". Audit: 140 reference-ish pages (largely current), 4 tutorials (three broken — deprecated SDK), 11 how-tos, dozens of fused pages. Actions: one tutorial rebuilt and CI-tested, two deleted; ticket analysis yielded a top-12 how-to list ("verify webhook signatures", "handle rate limits" — none existed); 30 fused pages split with cross-links; sidebar rebuilt around five user goals (Authenticate / Accept payments / Handle events / Test / Go live) replacing the six-team org-chart structure; 41 pages deleted. Eight weeks later: docs-answerable tickets ~25 → ~9/week, zero-result searches down 70%, and the "webhook retry" how-to — written from ticket transcripts — became the site's #3 page. Placement question into the PR template; quadrant drift since: two pages, caught in review.

## Related skills

- `api-documentation` — the reference quadrant's deep practice.
- `readme-authoring` — the front door that routes into this architecture.
- `onboarding-doc-design` — the same audience-journey thinking, internal edition.
- `architecture-decision-record` — raw material for the explanation quadrant.
