---
name: release-notes
description: >
  Use when announcing a release to its audience — translating the changelog
  into benefit-framed notes per audience (end users, admins, developers),
  choosing what to highlight and what to omit. Triggers: "write release
  notes", "announce this release", "what's new post", "customer-facing
  release summary", "in-app what's new".
---

# Release Notes

## When to use this skill
- A release needs announcing beyond the changelog: email, in-app panel, blog, customer bulletin.
- Marketing/support asked "what's in this release, in human terms?"
- NOT for the factual per-change record — that's `changelog-writing`, and it comes first; release notes are the *edited story* told on top of that record.

## Prerequisites
- The finished changelog for the release (the source of truth this document curates).
- The audience and channel named: end users in-app, admins by email, developers on the blog — they get different documents, not one compromise.

## Workflow

1. **Pick the 1–3 items that matter and lead with the best one.** Release notes are an attention transaction, not an inventory — the changelog already exists for completeness. Selection test per candidate: would a member of *this audience* change behavior or feel relief knowing this? The 14-entry changelog usually yields 2 headline items, one "also improved" cluster, and silence about the rest.

2. **Frame each highlight as the reader's outcome, then the feature:** benefit → what it is → how to get to it. "Find any invoice in seconds — search now covers PDF contents. Try it: press / and type." Not "Added full-text PDF indexing to the search subsystem." The changelog line describes the change; the release note sells the moment of use. Screenshots/GIFs for UI changes — one good visual outperforms every paragraph.

3. **Match register to audience without losing precision:** end users get outcomes and zero internals ("exports no longer time out on large workspaces"); admins get operational specifics (new permissions, migration windows, default changes — the things that page THEM); developers get the technical meat with links into `api-documentation` and the changelog. Cross-audience releases = multiple short documents from one changelog, each honest, none padded.

4. **Never bury the action-required items, whatever the tone:** breaking changes, deprecations with dates, new defaults, required migrations — flagged visually, top-adjacent, with the migration link (`changelog-writing` step 3's content, re-aimed at this audience). Delight-toned notes that hide the breaking change convert goodwill into betrayal precisely among your most engaged readers — the ones who actually read release notes.

5. **Write fixes with dignity when they earn a mention:** a widely-felt bug fixed deserves plain acknowledgment ("Fixed the sync error many of you hit after the June update — thanks for the reports") — it converts frustration into loyalty. Minor fixes cluster into one line with a changelog link. What's forbidden: "various improvements and bug fixes" as the *entire* notes (the null document that trains readers to stop reading), and euphemism around incidents readers experienced.

6. **Keep the cadence-appropriate shape:** weekly/continuous shippers — short notes, only when there's something to say (silence beats filler; batch small weeks into a monthly roundup); big versioned releases — the fuller treatment with sections per audience concern. Either way: dated, versioned, archived somewhere linkable, and consistent enough that readers know what to expect.

7. **Before shipping, run the two checks:** the support check — "will this generate questions we should preempt?" (new UI: where did the old button go; new default: how do I revert — answer them in the notes); and the honesty check against the changelog — nothing claimed that didn't ship, nothing material to this audience omitted. Support and docs teams see the notes *before* users do; they're the first responders to whatever the notes cause.

## Common pitfalls
- Publishing the changelog AS the release notes — complete, true, and unread by the audience that needed the story.
- Marketing voice untethered from the artifact ("reimagined from the ground up!" for a settings-page restyle) — one inflated claim discounts every future note.
- The buried lede: three paragraphs of feature celebration, then "additionally, API v1 sunsets next month" in the footer. Action-required floats to the top, always (step 4).
- One document for every audience — end users scrolling past migration SQL, admins hunting operational details inside feature marketing. Split them (step 3).
- Announcing by internal codename or team framing ("the Falcon pipeline is now GA") — readers own outcomes, not your org chart.
- Skipping notes for "boring" releases that quietly changed a default — the support tickets arrive anyway, now without a document to point at.

## Example
B2B product, quarterly release: changelog has 23 entries. Audience split produced two documents. Customer email (end users + admins): headline = report builder (benefit-framed, one GIF, "open Reports → New"); second item = the export-timeout fix, acknowledged plainly (top-5 support complaint all quarter); action-required block up top for admins — SSO session default shortened from 30d to 7d, with the revert setting linked (the step-7 support check added that revert line; it pre-empted the predictable #1 question). Developer blog post: the API rate-limit headers change with code samples, deprecation timeline for two v1 endpoints, changelog linked throughout. Not mentioned anywhere: 15 entries of internal fixes — present in the changelog for the record, absent from the story. Measured aftermath: the SSO default change — the release's riskiest item — generated 4 tickets instead of the forecast dozens, each answered by pasting the notes' revert link; the notes did the support team's week for them.

## Related skills
- `changelog-writing` — the factual record this document curates from.
- `release-versioning` — the release process both documents ride.
- `stakeholder-update` — the internal sibling with the same lead-with-what-matters discipline.
- `natural-prose-editing` — de-flattening the prose before it ships.
