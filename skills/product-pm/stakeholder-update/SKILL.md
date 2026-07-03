---
name: stakeholder-update
description: >
  Use when writing recurring status updates for stakeholders — progress,
  risks, and asks on one screen, readable in 90 seconds, honest about
  trouble early. Triggers: "status update for", "weekly update", "exec
  update", "project status report", "keep leadership informed".
---

# Stakeholder Update

## When to use this skill
- A project needs recurring status communication (weekly/biweekly) to people not in its daily flow.
- Updates exist but nobody reads them, or leadership keeps getting surprised anyway.
- NOT for incident communications (`incident-triage` step 6's clock-driven cadence) or roadmap presentations (`roadmap-communication`).

## Prerequisites
- Clarity on who reads this and what decisions they can make — the update's asks section is aimed at THEIR powers (budget, unblocking, prioritization calls).

## Workflow

1. **Lead with status-against-goal in one line, color-coded honestly:** "🟡 Checkout revamp — trending 1 week late on the June 30 target; mitigation in flight (below)." The reader's first question is "should I worry?"; answer it in the first ten words. Color semantics published once and kept stable: green = on track for the stated goal; yellow = at risk, mitigation exists; red = will miss without intervention — *red means "I need something", not "I failed"*, and the team's culture around that sentence determines whether you ever get true yellows.

2. **Structure the body as Progress / Risks / Asks — one screen, always the same shape:**
   - **Progress:** 3–5 outcome bullets since last update, tied to the goal ("payment-provider integration passing E2E" — not "had productive syncs"). Include a number where one exists (`metric-definition` energy: metric, target, current).
   - **Risks:** each with trajectory and owner ("vendor API latency 2× spec — testing workaround, decision by Fri — @sam"). A risk repeated three updates unchanged is a stalled risk: escalate or close it.
   - **Asks:** specific, addressed, dated ("need staffing decision on the second backend eng by Wed — blocks the July milestone"). No asks for three straight updates usually means unasked asks, not smooth sailing.

3. **Surface bad news early, framed as trajectory + plan:** the update where slippage first *appears* should be the update where it first *became likely* — not after it's certain. "Risk: migration running 40% slower than planned; if this week's fix fails, June 30 slips ~1 week; decision point Friday" costs a moment of discomfort and buys the thing updates exist for: no surprises. Bad news aged two weeks converts from information into betrayal.

4. **Write for the skimming reader:** bold the load-bearing phrases, one screen hard limit, links for depth (dashboard, PRD, incident doc) instead of inlined detail, and zero project-internal jargon or codenames without gloss (the exec forwarding your update to THEIR boss is the real audience — `roadmap-communication` step 4's "write as if forwarded" rule applies fully).

5. **Keep the goal visible and version its changes:** restate the goal + date in the header every time (readers forget; drift hides); if the goal itself changed, that's the headline with the reason — a goal quietly restated to match reality is the most corrosive update pattern there is.

6. **Make the cadence self-enforcing:** same day, same channel, every cycle — the metronome IS the trust signal (a skipped update reads as "hiding something" precisely when things are hard, which is when it's most tempting to skip); keep a running-notes file so the update is 15 minutes of curation, not an hour of archaeology (`meeting-notes-actions` and `weekly-review` feed it); a light template beats prose-from-scratch for consistency and skimmability both.

7. **Audit yours quarterly with the two questions:** did any reader get surprised by something this update stream should have carried? (each surprise = a step-3 failure to fix); and are the asks getting answered? (unanswered asks = wrong audience, wrong specificity, or the update isn't being read — diagnose which; an unread update is a writing problem before it's a reader problem).

## Common pitfalls
- The activity log: "met with design, discussed API, reviewed PRs" — motion without position. Readers need where-we-are-versus-goal, not what-we-did.
- Watermelon status: green outside, red inside, until the week before the deadline. One detonated watermelon costs more credibility than a year of honest yellows (step 1's culture note is the prevention).
- Burying the ask in paragraph four, unaddressed and undated — then noting "leadership never unblocks us." Asks are the update's payload; they go in their own section, named and dated.
- Length creep: the update that grew to four screens gets skimmed at best, and the risk paragraph on screen three might as well not exist. One screen; links carry depth.
- Same update to the exec and the working team — one gets drowned, the other starved. Two audiences = two updates (they share a source; they don't share a text).
- Ghost cadence: updates when there's "something to report" — which trains readers that silence is ambiguous. The metronome (step 6), even for "steady, no change" weeks.

## Example
Platform-migration project, biweekly exec update, previously a 3-page prose email nobody confirmed reading. Rebuilt on the template: header restated goal+date; status 🟡 in week 4 the moment the data-backfill rate first trended 40% under plan (step 3 — flagged at *likely*, not certain), with the Friday decision point named. The ask ("need the DBA on loan for two weeks — blocks cutover rehearsal") addressed to the one exec who could approve it, dated — approved in a day, versus the prior project where the same ask lived unnoticed in paragraph six for a month. Week 6: fix landed, back to 🟢, noted in one line. Post-project audit: zero exec surprises across 14 updates (the prior migration had produced three escalations from surprise alone); the update's running-notes file doubled as the retro's timeline. Total writing cost per update after the template settled: ~20 minutes.

## Related skills
- `roadmap-communication` — the strategic-horizon sibling.
- `meeting-notes-actions` — the raw material feed.
- `incident-triage` — the crisis-mode communication variant.
- `weekly-review` — the personal practice that makes updates cheap.
