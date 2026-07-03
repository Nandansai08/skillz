---
name: ui-heuristic-review
description: >
  Use when evaluating a UI without user testing — a structured Nielsen-
  heuristics pass over real task flows, producing severity-rated findings.
  Triggers: "review this UI", "heuristic evaluation", "usability review of
  this screen", "critique this flow", "why does this feel clunky",
  "UX audit".
---

# UI Heuristic Review

## When to use this skill
- A screen/flow needs expert evaluation before (or instead of, when budget forces it) user testing.
- Triaging "the UI feels off" complaints into fixable findings.
- NOT a replacement for watching real users — heuristics catch violations of known principles; they don't catch wrong mental models (only users reveal those). Say so in the readout.

## Prerequisites
- Access to the working UI (or high-fidelity prototype) and 2–4 realistic tasks to walk — reviewing screens without tasks reviews decoration.
- The intended user's context (expertise, frequency of use, device) — a violation for novices may be fine for daily expert users.

## Workflow

1. **Walk the tasks as the user first, heuristics second:** perform each task start-to-finish, narrating friction as you hit it, *before* opening the checklist. Task-first ordering finds the cross-screen problems (lost context between steps, inconsistent vocabulary across a flow) that per-screen checklist passes structurally miss. Note timestamps/screenshots as you go — findings need evidence.

2. **Then sweep with the ten heuristics, using the working questions:**
   - **Visibility of system status:** after every action, does something confirm it happened? (The silent save, the un-spinnered wait ≥1s.)
   - **Match to the real world:** whose vocabulary is on screen — the user's or the schema's? ("Sync entity mappings" vs "Connect your calendar".)
   - **User control & freedom:** is there undo? Can every state be exited? (The modal with no cancel; the wizard with no back.)
   - **Consistency & standards:** internal (same action, same word, everywhere) and external (does this control behave as the platform trained users to expect?).
   - **Error prevention:** are the mistakes designable-away? (Confirmation for destructive acts, constraints instead of validation — `form-design`'s territory.)
   - **Recognition over recall:** must the user remember anything across screens? (The code shown on step 2, demanded on step 5.)
   - **Flexibility & efficiency:** do frequent users get accelerators (shortcuts, bulk actions, recents)?
   - **Aesthetic & minimalist design:** does every element compete for attention proportionally to its importance?
   - **Error recovery:** are error messages diagnosis + way forward, in human language? (`microcopy-writing` owns the wording.)
   - **Help & documentation:** is help contextual and task-shaped, when it exists?

3. **Record each finding as evidence + principle + severity:** what happened (screenshot, step), which heuristic, and severity on the standard scale — 0 not-a-problem / 1 cosmetic / 2 minor (annoys, doesn't block) / 3 major (delays or misleads significantly, users will hit it often) / 4 catastrophic (blocks task or destroys data). Severity = frequency × impact × persistence (will it keep hurting after learning?). The scale is what keeps the readout from being a taste debate.

4. **Multiply the evaluators when stakes justify it:** one expert catches ~35% of issues; 3–5 independent passes (THEN merged) catch most of them. Independence matters — a group walkthrough converges on the loudest reviewer's findings. Solo review is fine for routine work; say its coverage limit in the readout.

5. **Separate violations from taste, ruthlessly:** "the primary action is visually quieter than three secondary ones" is a finding (heuristic 8, evidence attached); "I'd use a different blue" is not. Every finding must name its principle and its user cost — anything that can't gets cut or explicitly labeled opinion. This discipline is what makes design review consumable by non-designers.

6. **Deliver as a ranked fix-list, not a violations museum:** sort by severity, cluster by root cause (twelve label inconsistencies = one finding: "no shared vocabulary — fix the term sheet, not twelve strings"), pair each major finding with a direction (not a full redesign — "expose the sync status inline; pattern exists on the billing page"), and lead the readout with the 3–5 that matter. Findings priced by fix-effort where obvious helps the triage meeting (`prioritization-frameworks` energy at review scale).

7. **Close the loop against reality:** the majors should reconcile with support tickets and analytics (rage-clicks, drop-off points) where they exist — heuristic findings corroborated by funnel data jump the fix queue; and the recurring finding categories feed the design system and review checklist so the same violation stops being rediscovered quarterly (`design-system-tokens` and component guidelines are where findings go to become prevention).

## Common pitfalls
- Screen-by-screen review without tasks: every screen individually defensible, the flow across them broken — the most common gap in checklist-driven reviews (step 1 exists for this).
- The 60-finding readout with no severities: the team fixes the eight easy cosmetic items, declares the review addressed, and the severity-4 data-loss modal survives. Rank or it didn't happen.
- Taste wearing a heuristic costume ("violates minimalism" = "I don't like it") — the principle+cost test (step 5) is the filter.
- Reviewing as yourself instead of the user context: keyboard-shortcut findings for a once-a-quarter admin tool; hand-holding findings for a daily expert tool. Prerequisites exist to be used.
- Fix-prescribing at full fidelity ("redesign this as a stepped wizard with...") — the review names problems and directions; over-specified fixes get defensively rejected and couple the finding's validity to the fix's.
- No self-check against data: three "major" findings on a flow whose analytics show zero drop-off deserve a humility pass — sometimes users aren't hitting what experts trip on (step 7's reconciliation cuts both ways).

## Example
Review target: invoice-creation flow, prompted by "feels clunky" churn-call mentions. Two evaluators, independent passes, three tasks (create, duplicate-and-edit, fix a rejected invoice). Task walk found the flow's worst issue before any checklist: after submitting, users land on the list view with NO status indicator — the invoice silently enters "processing" and users resubmitted duplicates (heuristic 1, severity 4 — corroborated: support's #2 ticket category was duplicate invoices; analytics showed 9% resubmit rate). Checklist sweep added: destructive "discard draft" with no confirm (sev 3), vocabulary split — "client" on screen 1, "customer" on screens 2–4, "account" in errors (clustered as one sev-2 root-cause finding: term sheet), and a taste-labeled note on crowded toolbar (opinion, flagged as such). Readout: 14 findings clustered to 8, top 3 led. The status-visibility fix (inline processing badge + disabled resubmit, pattern borrowed from the payments page) shipped in a week: resubmit rate 9% → 0.4%, duplicate-invoice tickets down 80%. The term sheet became a design-system PR — the review's finding converted into prevention.

## Related skills
- `accessibility-audit` — the parallel structured pass this review doesn't cover.
- `form-design` — deep-dive when the findings concentrate in forms.
- `microcopy-writing` — fixing the words the review flagged.
- `empty-loading-error-states` — the states reviews most often find missing.
