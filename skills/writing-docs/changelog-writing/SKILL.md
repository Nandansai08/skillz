---
name: changelog-writing
description: >
  Use when writing user-facing changelogs — translating commits into
  impact statements readers care about, structuring by change type,
  flagging breaking changes loudly. Triggers: "write the changelog",
  "changelog entry for this release", "turn these commits into a
  changelog", "CHANGELOG.md", "what's changed section". NOT for the
  versioning/tagging mechanics (see release-versioning) or the narrative
  marketing announcement (see release-notes).
---

# Changelog Writing

## Overview

A changelog is an edited publication, not a git query — every entry answers "does this affect me?", breaking changes get the full migration treatment, and the six fixup commits merge into the feature they belong to. The upgrade-decision skim is its one job.

## When to Use

- Preparing a release's changelog for a library, API, app, or internal platform.
- Auditing a changelog that's become an unread commit-log mirror.

**When NOT to use:**
- Version/tag mechanics — `release-versioning`.
- The announcement narrative — `release-notes` (the changelog is the factual record it curates from).

## Prerequisites

- The release's actual diff/commit range and knowledge of *who reads this* — library consumers, end users, and platform teams read for different stakes.

## The Workflow

1. **Write every entry as reader-impact, not author-activity.** "Refactored session handling" (author diary) → "Fixed: sessions no longer expire during active use (#1204)". Test per line: can a consumer decide *whether this affects them* from the line alone? Failing lines get rewritten or cut.

2. **Use the keep-a-changelog sections, in severity order:** `Breaking` → `Security` → `Added` → `Fixed` → `Deprecated` → `Removed`. Fixed order means readers learn where to look; severity-first means the expensive-to-miss items can't hide below thirty `Fixed` entries.

3. **Give breaking changes the full treatment — they're the reason changelogs exist:** what broke, who's affected, before/after, and the migration path, inline:
   ```markdown
   ### Breaking
   - `client.connect()` no longer retries automatically. If you relied on
     built-in retries, wrap calls: `client.connect(retry=RetryPolicy(...))`.
     Affects: anyone calling connect() without their own retry handling.
   ```
   A breaking change described in one vague line converts directly into support tickets and burned upgrade trust (`dependency-upgrade` step 1 is your reader).

4. **Curate ruthlessly from the raw commit list:** merge the six "fix typo/address review" commits into their feature; cut pure-internal changes unless the reader-visible effect exists ("builds ~2× faster"); translate ticket-speak (`Fixed JIRA-4412` → what 4412 *was*, linked). The editing is the entire difference between a changelog that's read and one that isn't.

5. **Anchor entries with links and credits:** PR/issue links per entry, CVE IDs on security fixes (severity + affected surface + urgency; not exploit recipes pre-disclosure, not "security improvements" forever), contributor credit on external contributions.

6. **Keep the mechanical contract:** newest first, every release dated (ISO), an `Unreleased` section accumulating entries *as PRs merge* (changelog entry as PR checklist item is the enforcement — release-day archaeology is the process failure this prevents), version headings linked to diffs.

7. **Read the finished section as the upgrade decision it is:** a consumer skims asking "should I take this, and what must I do?" If that's not answerable in 30 seconds — breaking changes unmissable, migrations inline, the rest scannable — restructure. That skim IS the changelog's one job.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "The git log is the changelog — it's all there" | Forty entries of 'wip' and 'address review' is informationally void, and it trains readers to never look again — which un-reads the one entry that mattered. |
| "The breaking change is listed — entry #14, under Changed" | Buried mid-list in neutral voice is how the most important line in the release gets missed. Breaking changes get the siren section, the migration, and the affects-statement (step 3). |
| "'Various bug fixes and improvements' covers the small stuff" | The null entry: if it mattered enough to ship, it merits naming; if not, cut the line. Empty calories teach readers the whole document is empty. |
| "I'll write it release day from memory" | Memory misses the subtle breaking change from three weeks ago that its author would have flagged at merge time. The Unreleased-section discipline (step 6) makes release day a 20-minute edit. |
| "Internal codenames are fine — our users know us" | 'The Falcon pipeline is now GA' means nothing outside the org chart. Readers own outcomes, not your project names. |
| "Detailing the security fix helps attackers" | And 'security improvements' forever helps nobody assess urgency. The honest pattern — severity + surface + upgrade urgency, CVE linked when public — serves defenders without writing the exploit. |

## Red Flags

- Entries readable only by people who attended the standups.
- Breaking changes outside the Breaking section, or without migrations.
- A changelog whose entry count equals the commit count.
- No dates, no diff links, no Unreleased section.
- "Various fixes" as a recurring entry.
- Consumers filing "X broke" issues for documented-nowhere changes.

## Verification

- [ ] Every entry passes the does-this-affect-me test — spot-check five.
- [ ] Breaking changes: before/after + migration + affects-statement, top section — verified.
- [ ] Entry count meaningfully below commit count (curation happened) — numbers noted.
- [ ] Links per entry; dates and diff links per release.
- [ ] Unreleased-section flow active (changelog line in the PR template) — link.
- [ ] The 30-second upgrade-decision skim performed by someone else — confirmed.

## Example

Library release 3.2.0, raw material: 61 commits. Curation: 61 → 14 entries. The one that mattered: a commit titled "normalize header casing" was, on reading the diff, breaking for anyone reading `response.headers` with case-sensitive keys — promoted to the Breaking section with before/after and a one-line migration, affects-statement included. Six review-fixup commits merged into their features; "bump internal test matrix" cut; two community PRs credited by handle. The Unreleased-section discipline adopted afterward: entries now written by PR authors at merge time, and the 3.3.0 changelog took 20 minutes instead of an afternoon of `git log` forensics. Measured effect: zero "3.2 broke my header code" issues filed — the two users who WERE affected both referenced the migration line in their upgrade PRs.

## Related skills

- `release-versioning` — the version/tag machinery this rides on.
- `release-notes` — the narrative layer for announcement channels.
- `dependency-upgrade` — the consumer-side reader this document serves.
- `api-documentation` — where the API changelog variant lives.
