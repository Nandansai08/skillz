---
name: deep-work-scheduling
description: >
  Use when engineering time for focused work — timeblock design, interrupt
  budgets, context-switch costs, defending blocks in a meeting-heavy
  calendar. Triggers: "no time to focus", "calendar is all meetings",
  "timeblocking", "protect focus time", "context switching is killing me",
  "schedule deep work".
---

# Deep Work Scheduling

## When to use this skill
- Meaningful work keeps losing to meetings and interrupts; output happens only at night or not at all.
- Designing a personal (or team-level) focus-time system.
- NOT for what to work ON (`weekly-review` step 5 supplies the priorities) — this is the time-mechanics that give those priorities hours.

## Prerequisites
- One honest week of data: where time actually went (calendar audit + a rough interrupt tally). Scheduling reform without the audit fixes the imagined problem.

## Workflow

1. **Audit the week and name the real enemy:** categorize the calendar (deep work / meetings-that-needed-you / meetings-that-didn't / reactive-interrupt time), and tally interrupts by source for 2–3 days (Slack, drive-bys, alerts, self-inflicted tab-switching — the last one is routinely a third of the total, and it's the one nobody blames). The audit's typical findings: less meeting-load than believed, more fragmentation than believed — 6 "free" hours in 45-minute shards is zero deep-work hours, because the shard below ~90 minutes rarely reaches depth at all (`complexity-analysis` for your attention: the fixed cost of loading context dominates short blocks).

2. **Place 2–3 blocks of 2–4 hours at your biological peak, recurring:** identify when your focus is naturally best (most people: morning) and put the blocks THERE as recurring calendar events — protecting your worst hours is theater. Realistic dose for most roles: 2–4 deep blocks/week reliably beats an aspirational daily block abandoned by Wednesday; blocks are appointments with the week's top priority (`weekly-review` step 5 assigns the tenant — an empty-purpose block is the first one surrendered).

3. **Make the blocks structurally defensible:** named calendar events (auto-decline where culture permits), Slack status + notifications actually off (the status that isn't enforced by the notification settings is decoration), phone elsewhere, ONE work-surface open. The context-switch math justifies the rigor: each "quick check" costs the reload of working state — a 30-second glance with a 15-minute re-immersion tax means four glances converts a deep block into shallow shards (the interruption's cost is never the interruption's duration).

4. **Pair every closed door with an open one — the interrupt budget:** published availability (office hours, "async always, sync after 2pm", the team's on-call-for-questions rotation) is what makes block-defense socially sustainable — colleagues tolerate unavailability that's bounded and predictable, and revolt at unavailability that's ambient. The genuinely-urgent channel stays open and NAMED ("page me via X for prod issues") — a focus system with no escape hatch gets bypassed entirely the first real fire, then routinely (`human-in-the-loop-checkpoints` shares this design instinct: the override path must exist and be narrow).

5. **Batch the shallow work into deliberate shards:** email/Slack processed in 2–3 scheduled passes instead of ambient monitoring; small tasks (approvals, reviews under 15 minutes, expense clicks) collected into an "admin block" (the same batching logic as `ci-pipeline-design`'s fail-fast ordering — cheap items grouped, not interleaved with expensive ones); meetings clustered where calendar control permits (two meeting-dense afternoons beat five polka-dotted days — meeting adjacency is nearly free; meeting scatter is what shreds the week).

6. **Enter blocks warm and exit them clean:** a 2-minute entry ritual (yesterday's ending note tells you exactly where to resume — the "start where?" flail is a 20-minute tax on every cold block) and an exit note when the block ends mid-thought ("next: the retry test, the branch is mid-refactor at the pool logic" — future-you's warm start). The entry/exit notes are the cheapest productivity artifact in this entire skill.

7. **Review the system monthly with two numbers:** blocks-held rate (scheduled vs actually-defended — chronic surrender means the blocks are mis-placed, over-dosed, or the interrupt budget's open-door isn't credible yet) and the fragmentation trend (longest unbroken stretch per week). Team-level version: shared focus windows (org-wide no-meeting mornings outperform individual heroics — fragmentation is substantially a coordination problem, and coordinated quiet is its coordinated fix).

## Common pitfalls
- Scheduling blocks without defending them: the calendar says Deep Work, the notifications say otherwise — step 3's enforcement is the block; the event is just its label.
- Protecting the wrong hours: focus blocks at 4pm because mornings "fill up" — the peak hours are the asset; defend those, surrender the trough (step 2).
- The no-escape-hatch system: total unavailability, bypassed at the first genuine urgency, then discredited entirely — the named urgent channel (step 4) is what makes the rest hold.
- Blaming meetings while self-interrupting: the audit's uncomfortable third — tab-switching and ambient Slack are self-inflicted fragmentation no calendar reform touches (step 1's tally names it).
- Shard-sized "focus time": six 45-minute gaps counted as six hours of capacity — below the depth threshold, they're shallow-work slots wearing focus labels (step 1's math).
- Heroic over-dosing: daily 4-hour blocks scheduled, abandoned by week two, system discredited — the sustainable dose (step 2) held for months beats the aspirational one held for days.

## Example
Senior engineer, complaint: "zero focus time, calendar is meetings." Audit's actual findings: 14 meeting-hours (not the believed 25), but no unbroken stretch over 50 minutes all week — and the interrupt tally: 31/day, of which 12 were self-inflicted check-ins. System built: two 3-hour morning blocks (Tue/Thu, peak hours, recurring, auto-decline) + one 90-minute Friday block; tenant assigned each Friday from the weekly review. Defense: notifications off via OS-level focus mode (the Slack status alone had already failed), urgent channel named (pager for prod, and the team told exactly that). Open door: daily 2–4pm office hours published — which is what made the mornings socially survivable; the one colleague who'd tested the boundary switched to the office hours within two weeks. Shallow batching: Slack at 11:30/2/4:30, admin block Friday. Entry/exit notes adopted after the first cold-start flail. Month-two numbers: blocks-held 10 of 12; longest stretch 3h (from 50min); the two surrendered blocks both traced to a genuinely urgent incident — the escape hatch used as designed, system's credibility intact. The migration project that had crawled for a quarter shipped in five weeks of Tuesday-Thursdays — the before/after that converted two teammates, whose adoption then birthed the team's shared no-meeting mornings.

## Related skills
- `weekly-review` — assigns each block's tenant.
- `meeting-notes-actions` — makes the meetings that survive worth their slots.
- `interview-prep-technical` — a consumer of these blocks with a deadline.
- `alerting-design` — the same actionability discipline, applied to your interrupts.
