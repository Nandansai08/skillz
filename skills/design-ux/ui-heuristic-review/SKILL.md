---
name: ui-heuristic-review
description: >
  Use when evaluating a UI without user testing — a structured
  Nielsen-heuristics pass over real task flows, producing severity-rated
  findings. Triggers: "review this UI", "heuristic evaluation",
  "usability review of this screen", "critique this flow", "why does this
  feel clunky", "UX audit". NOT a replacement for watching real users —
  heuristics catch violations of known principles; only users reveal
  wrong mental models.
---

# UI Heuristic Review

## Overview

Heuristic review is tasks-first (cross-screen problems are invisible to per-screen checklists), principle-anchored (every finding names its heuristic and its user cost — or it's taste), and severity-ranked (or the team fixes eight cosmetic items while the data-loss modal survives).

## When to Use

- A screen/flow needs expert evaluation before (or when budget forbids) user testing.
- Triaging "the UI feels off" complaints into fixable findings.

**When NOT to use:**
- As a substitute for user research on mental models — say the coverage limit in the readout.

## Prerequisites

- Access to the working UI (or hi-fi prototype) and 2–4 realistic tasks to walk — reviewing screens without tasks reviews decoration.
- The intended user's context (expertise, frequency, device) — a violation for novices may be fine for daily experts.

## The Workflow

1. **Walk the tasks as the user first, heuristics second:** perform each task start-to-finish, narrating friction, *before* opening the checklist. Task-first ordering finds the cross-screen problems (lost context between steps, inconsistent vocabulary across a flow) that per-screen passes structurally miss. Screenshots/timestamps as you go — findings need evidence.

2. **Then sweep with the ten heuristics, using the working questions:**
   - **Visibility of system status:** after every action, does something confirm it happened? (The silent save; the un-spinnered wait ≥1s.)
   - **Match to the real world:** whose vocabulary is on screen — the user's or the schema's?
   - **User control & freedom:** undo? Can every state be exited?
   - **Consistency & standards:** internal (same action, same word) and external (platform conventions).
   - **Error prevention:** constraints over validation; confirms for destructive acts (`form-design`).
   - **Recognition over recall:** must the user remember anything across screens?
   - **Flexibility & efficiency:** accelerators for frequent users?
   - **Aesthetic & minimalist design:** does attention-competition match importance?
   - **Error recovery:** messages = diagnosis + way forward, human language (`microcopy-writing`).
   - **Help & documentation:** contextual and task-shaped, when it exists.

3. **Record each finding as evidence + principle + severity:** what happened (screenshot, step), which heuristic, and severity 0–4 (0 not-a-problem / 1 cosmetic / 2 minor / 3 major — delays or misleads, hit often / 4 catastrophic — blocks task or destroys data). Severity = frequency × impact × persistence. The scale is what keeps the readout from being a taste debate.

4. **Multiply the evaluators when stakes justify it:** one expert catches ~35% of issues; 3–5 INDEPENDENT passes then merged catch most (a group walkthrough converges on the loudest voice). Solo review is fine for routine work; state its coverage limit.

5. **Separate violations from taste, ruthlessly:** "the primary action is visually quieter than three secondary ones" is a finding (heuristic 8, evidence attached); "I'd use a different blue" is not. Every finding names its principle and user cost — anything that can't gets cut or labeled opinion. This is what makes review consumable by non-designers.

6. **Deliver as a ranked fix-list, not a violations museum:** sort by severity, cluster by root cause (twelve label inconsistencies = ONE finding: "no shared vocabulary — fix the term sheet"), pair majors with a direction, not a full redesign ("expose sync status inline; pattern exists on the billing page" — over-specified fixes get defensively rejected). Lead with the 3–5 that matter.

7. **Close the loop against reality:** majors should reconcile with support tickets and analytics (rage-clicks, drop-offs) — corroborated findings jump the queue, and three "majors" on a flow with zero drop-off deserve a humility pass (users aren't always hitting what experts trip on). Recurring finding categories feed the design system so the same violation stops being rediscovered quarterly.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "I'll review each screen — thorough and systematic" | Screen-by-screen finds per-screen defects while the flow across them stays broken — the most common gap in checklist reviews. Tasks first; the checklist sweeps after. |
| "Sixty findings shows a rigorous review" | Unranked volume means the team fixes the eight easy cosmetics and declares victory while the severity-4 survives. Ranked-and-clustered or the review changes nothing. |
| "It violates minimalism (I don't like the layout)" | Taste in a heuristic costume. The principle + user-cost test is the filter — findings that can't pass it are opinions, labeled or cut. |
| "As an expert user, this flow feels fine to me" | The prerequisite context exists because you aren't the once-a-quarter admin this tool serves. Review as the defined user or review yourself. |
| "I'll include the full redesign for each finding" | Over-specified fixes couple the finding's validity to the fix's, and get defensively rejected together. Name the problem and a direction; design happens after. |
| "Analytics show no drop-off, but the review found majors — ship the fixes anyway" | Corroboration cuts both ways (step 7): expert-tripped ≠ user-hit. The humility pass keeps heuristic review honest about its false-positive rate. |

## Red Flags

- A review with no task walkthrough evidence.
- Findings without heuristic names or severities.
- Twelve variants of the same root cause listed separately.
- The readout's top item being cosmetic.
- Group-session "independent" evaluation.
- Findings never reconciled against tickets/analytics.

## Verification

- [ ] Tasks walked end-to-end with evidence captured — artifacts attached.
- [ ] All ten heuristics swept — coverage visible in the findings' principle tags.
- [ ] Every finding: evidence + principle + 0–4 severity; taste items labeled or absent.
- [ ] Findings clustered by root cause; count before/after clustering noted.
- [ ] Readout leads with top 3–5; directions (not full redesigns) attached to majors.
- [ ] Reality reconciliation done (tickets/analytics cross-check) — noted per major.

## Example

Review target: invoice-creation flow, prompted by "feels clunky" churn-call mentions. Two evaluators, independent passes, three tasks. Task walk found the worst issue before any checklist: after submitting, users land on the list view with NO status indicator — the invoice silently enters "processing" and users resubmitted duplicates (heuristic 1, severity 4 — corroborated: support's #2 ticket category was duplicate invoices; 9% resubmit rate in analytics). Checklist sweep added: destructive "discard draft" with no confirm (sev 3), vocabulary split — "client"/"customer"/"account" across the flow (clustered as one sev-2 root-cause finding: term sheet), and a taste-labeled note on toolbar crowding. Readout: 14 findings clustered to 8, top 3 led. The status-visibility fix (inline processing badge + disabled resubmit, pattern borrowed from payments) shipped in a week: resubmit rate 9% → 0.4%. The term sheet became a design-system PR — the finding converted into prevention.

## Related skills

- `accessibility-audit` — the parallel structured pass this review doesn't cover.
- `form-design` — deep-dive when findings concentrate in forms.
- `microcopy-writing` — fixing the words the review flagged.
- `empty-loading-error-states` — the states reviews most often find missing.
