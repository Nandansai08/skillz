---
name: changelog-writing
description: >
  Use when writing user-facing changelogs — translating commits into impact
  statements readers care about, structuring by change type, flagging
  breaking changes loudly. Triggers: "write the changelog", "changelog
  entry for this release", "turn these commits into a changelog",
  "CHANGELOG.md", "what's changed section".
---

# Changelog Writing

## When to use this skill
- Preparing a release's changelog for a library, API, app, or internal platform.
- Auditing a changelog that's become an unread commit-log mirror.
- NOT for the versioning/tagging mechanics (`release-versioning`) or the narrative marketing announcement (`release-notes` — the changelog is the factual record; release notes are the story told on top of it).

## Prerequisites
- The release's actual diff/commit range and, crucially, knowledge of *who reads this changelog* — a library's consumers, an app's end users, or internal platform teams read for different stakes.

## Workflow

1. **Write every entry as reader-impact, not author-activity.** The transformation rule: from "what we did" to "what changes for you." — "Refactored session handling" (author diary) → "Fixed: sessions no longer expire during active use (#1204)". Test per line: can a consumer decide *whether this line affects them* from the line alone? Entries failing that test get rewritten or cut.

2. **Use the keep-a-changelog sections, in severity order:** `Breaking` (or `Changed` with breaking flags) → `Security` → `Added` → `Fixed` → `Deprecated` → `Removed`. Fixed order means readers learn where to look; severity-first means the expensive-to-miss items can't hide below the fold of thirty `Fixed` entries.

3. **Give breaking changes the full treatment — they're the reason changelogs exist:** what broke, who's affected, the before/after, and the migration path, inline (not "see docs"):
   ```markdown
   ### Breaking
   - `client.connect()` no longer retries automatically. If you relied on
     built-in retries, wrap calls: `client.connect(retry=RetryPolicy(...))`.
     Affects: anyone calling connect() without their own retry handling.
   ```
   A breaking change described in one vague line converts directly into support tickets and burned upgrade trust (`dependency-upgrade` step 1 is your reader — write for that person).

4. **Curate ruthlessly from the raw commit list:** merge the six "fix typo/address review/wip" commits into the one feature they belong to; cut pure-internal changes (CI tweaks, test refactors) unless the reader-visible effect exists ("builds are ~2× faster"); translate ticket-speak (`Fixed JIRA-4412` → what 4412 *was*, with the ticket linked). The changelog is an edited publication, not a git query — that's the entire difference between one that's read and one that isn't (`release-versioning` step 6's "generated then edited").

5. **Anchor entries with links and credits:** PR/issue links per entry (the reader who IS affected needs the details one click away), CVE IDs on security fixes, and contributor credit on external contributions (cheap to give, meaningful to receive, visibly community-building for open source).

6. **Keep the mechanical contract:** newest release first, every release dated (ISO), an `Unreleased` section accumulating entries *as PRs merge* (the release-day archaeology of "what's in this one?" is the process failure this prevents — changelog entry as PR checklist item is the enforcement), and version headings linked to diffs (`[2.3.0]: https://github.com/org/repo/compare/v2.2.0...v2.3.0`).

7. **Read the finished section as the upgrade decision it is:** a consumer skims it asking "should I take this release, and what must I do?" If the answer isn't derivable in 30 seconds — breaking changes unmissable, migration steps inline, everything else scannable — restructure. That skim IS the changelog's one job.

## Common pitfalls
- The commit-log dump: 45 entries of "fix", "update deps", "address feedback" — technically complete, informationally void, trains readers to never look again.
- Breaking changes buried mid-list in neutral voice ("Changed connect() retry behavior") — the entry most needing a siren, written as a shrug.
- Writing for teammates instead of consumers: internal codenames, "the new pipeline", references to meetings. The reader has none of that context.
- The empty-calorie entry: "Various bug fixes and improvements." If it mattered enough to ship, it matters enough to name; if it doesn't merit naming, cut the line entirely.
- Changelog written release-day from `git log` memory — misses the subtle breaking change from three weeks ago that its author would have flagged at PR time (step 6's accumulate-as-you-merge).
- Security fixes described too specifically pre-disclosure or too vaguely forever ("security improvements") — the honest pattern: severity + affected surface + upgrade urgency, CVE linked when public.

## Example
Library release 3.2.0, raw material: 61 commits. Curation: 61 → 14 entries. The one that mattered: a commit titled "normalize header casing" was, on reading the diff, breaking for anyone reading `response.headers` with case-sensitive keys — promoted to the Breaking section with before/after and a one-line migration (`headers.get()` was already case-insensitive; direct dict access wasn't), affects-statement included. Six review-fixup commits merged into their features; "bump internal test matrix" cut; two community PRs credited by handle. The Unreleased-section discipline adopted afterward: entries now written by PR authors at merge time, and the 3.3.0 changelog took 20 minutes instead of an afternoon of `git log` forensics. Measured effect, imperfect but real: zero "3.2 broke my header code" issues filed — the two users who WERE affected both referenced the migration line in their upgrade PRs.

## Related skills
- `release-versioning` — the version/tag machinery this rides on.
- `release-notes` — the narrative layer for announcement channels.
- `dependency-upgrade` — the consumer-side reader this document serves.
- `api-documentation` — where the API changelog variant lives.
