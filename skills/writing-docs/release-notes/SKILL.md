---
name: release-notes
description: >
  Use when announcing a release to its audience — translating the
  changelog into benefit-framed notes per audience (end users, admins,
  developers), choosing what to highlight and what to omit. Triggers:
  "write release notes", "announce this release", "what's new post",
  "customer-facing release summary", "in-app what's new". NOT for the
  factual per-change record — that's changelog-writing, and it comes
  first.
---

# Release Notes

## Overview

Release notes are an attention transaction: 1–3 highlights framed as the reader's outcome, action-required items unmissable, and silence about the other fifteen entries (the changelog holds them). The support check before shipping is what converts the notes from announcement into deflection.

## When to Use

- A release needs announcing beyond the changelog: email, in-app panel, blog, customer bulletin.
- Marketing/support asked "what's in this release, in human terms?"

**When NOT to use:**
- The factual per-change record — `changelog-writing`, first; notes curate from it.

## Prerequisites

- The finished changelog for the release.
- The audience and channel named: end users in-app, admins by email, developers on the blog — different documents, not one compromise.

## The Workflow

1. **Pick the 1–3 items that matter and lead with the best one.** Selection test per candidate: would a member of *this audience* change behavior or feel relief knowing this? The 14-entry changelog usually yields 2 headline items, one "also improved" cluster, and silence about the rest — the changelog already exists for completeness.

2. **Frame each highlight as the reader's outcome, then the feature:** benefit → what it is → how to reach it. "Find any invoice in seconds — search now covers PDF contents. Try it: press / and type." Not "Added full-text PDF indexing to the search subsystem." Screenshots/GIFs for UI changes — one good visual outperforms every paragraph.

3. **Match register to audience without losing precision:** end users get outcomes and zero internals; admins get operational specifics (new permissions, migration windows, default changes — the things that page THEM); developers get the technical meat with links into `api-documentation`. Cross-audience releases = multiple short documents from one changelog.

4. **Never bury the action-required items, whatever the tone:** breaking changes, deprecations with dates, new defaults, required migrations — flagged visually, top-adjacent, migration linked inline. Delight-toned notes that hide the breaking change convert goodwill into betrayal precisely among the readers engaged enough to read notes.

5. **Write fixes with dignity when they earn a mention:** a widely-felt bug fixed deserves plain acknowledgment ("Fixed the sync error many of you hit after the June update — thanks for the reports") — it converts frustration into loyalty. Minor fixes cluster into one line with a changelog link. Forbidden: "various improvements and bug fixes" as the entire notes, and euphemism around incidents readers experienced.

6. **Keep the cadence-appropriate shape:** continuous shippers — short notes only when there's something to say (silence beats filler; batch small weeks into a monthly roundup); big versioned releases — the fuller per-audience treatment. Either way: dated, archived somewhere linkable, consistent.

7. **Before shipping, run the two checks:** the support check — "will this generate questions we should preempt?" (new UI: where did the old button go; new default: how do I revert — answer them IN the notes); and the honesty check against the changelog — nothing claimed that didn't ship, nothing material to this audience omitted. Support and docs see the notes *before* users do.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Publish the changelog as the notes — it's complete" | Complete, true, and unread by the audience that needed the story. The changelog is the record; the notes are the edited attention transaction on top of it. |
| "Mention everything — every team wants their item featured" | Fifteen items = zero highlights. The selection test (would THIS reader change behavior?) is the editorial spine; internal fairness is not a reader benefit. |
| "Lead with the exciting stuff; the deprecation can go at the bottom" | The buried action-required item is the support ticket factory and the trust burner. Whatever the tone, must-act floats to the top — always (step 4). |
| "Don't acknowledge the bug — it makes us look bad" | Your most engaged readers HIT the bug; plain acknowledgment converts their frustration into loyalty, euphemism converts it into contempt. Dignity, not spin. |
| "One document for all audiences — efficient" | End users scroll past migration SQL; admins hunt operational details inside feature marketing. One changelog, N short documents (step 3). |
| "It's a boring release — skip the notes" | The boring release that changed a default generates the tickets anyway, now with no document to point at. If a default moved, the notes exist (step 7's support check). |

## Red Flags

- The notes are the changelog, verbatim.
- A breaking change below the fold of feature celebration.
- "Reimagined from the ground up!" for a restyle.
- Internal codenames in customer-facing text.
- Support discovering the release from customer tickets.
- "Various improvements" as the entire body.

## Verification

- [ ] ≤3 highlights, each passing the audience selection test — listed with rationale.
- [ ] Action-required block top-adjacent with inline migration/revert links — visible.
- [ ] Per-audience versions produced where audiences diverge — links.
- [ ] Support check run: predicted questions answered in the notes — support sign-off noted.
- [ ] Honesty check against the changelog — nothing overclaimed/omitted; diff reviewed.
- [ ] Dated and archived at a linkable home.

## Example

B2B product, quarterly release: changelog has 23 entries. Audience split produced two documents. Customer email (end users + admins): headline = report builder (benefit-framed, one GIF, "open Reports → New"); second item = the export-timeout fix, acknowledged plainly (top-5 support complaint all quarter); action-required block up top for admins — SSO session default shortened from 30d to 7d, with the revert setting linked (the step-7 support check added that revert line; it pre-empted the predictable #1 question). Developer blog post: the API rate-limit headers change with code samples, deprecation timeline for two v1 endpoints, changelog linked throughout. Not mentioned anywhere: 15 entries of internal fixes — present in the changelog, absent from the story. Measured aftermath: the SSO default change — the release's riskiest item — generated 4 tickets instead of the forecast dozens, each answered by pasting the notes' revert link.

## Related skills

- `changelog-writing` — the factual record this document curates from.
- `release-versioning` — the release process both documents ride.
- `stakeholder-update` — the internal sibling with the same lead-with-what-matters discipline.
- `natural-prose-editing` — de-flattening the prose before it ships.
