---
name: technology-evaluation
description: >
  Use when evaluating a library, framework, or tool for adoption —
  maintenance signals, exit costs, spike design, comparing against the
  incumbent honestly. Triggers: "should we use X", "evaluate this
  library", "X vs Y for our stack", "is this project maintained", "build
  vs buy vs adopt", "picking a framework". NOT for surveying a research
  field (see literature-review) — this ends in an adopt/reject decision.
---

# Technology Evaluation

## Overview

Criteria written before the demo, vitals read from maintenance data (not stars), the exit priced before the entrance, and a spike aimed at YOUR risks instead of the tutorial's path. The incumbent competes as a candidate — and starts ahead, because migration costs are real and the challenger's are always underestimated.

## When to Use

- Choosing a dependency, platform, or tool with meaningful switching costs.
- Re-evaluating an incumbent aging badly (chronic CVEs, abandonment).

**When NOT to use:**
- Field surveys — `literature-review`.

## Prerequisites

- The requirement stated independently of any candidate — a requirement written after falling for a tool is a justification.
- The incumbent/status-quo named as a candidate on equal footing.

## The Workflow

1. **Write the evaluation criteria and weights BEFORE touching candidates:** must-haves (disqualifiers — license, platform, compliance), weighted wants (performance envelope, learning curve for THIS team, operational burden), and the context facts that bend everything (existing expertise, scale actually needed — not aspirational, deployment constraints). Criteria written after the demo inherit the demo's charisma.

2. **Shortlist to 2–3 + incumbent, quickly and defensibly:** the long list dies on must-haves and coarse signals in an afternoon; depth budget belongs to finalists. Log one-line eliminations — the "why not Z?" question WILL arrive (`architecture-decision-record` fairness applies).

3. **Read the maintenance vitals — the health check predicting your next three years:**
   - **Cadence & bus factor:** release recency AND contributor concentration (one person = one burnout from abandonment; are they paid to work on it?).
   - **Issue/PR hygiene:** median time-to-response (not count — popular projects have many issues; dead ones have many UNANSWERED).
   - **Security posture:** CVE response speed, security policy, signed releases.
   - **Ecosystem gravity:** who else depends on it (big dependents = shared fate); answer density for the errors you'll hit.
   - **Governance & money:** foundation/company/hobby (whose roadmap wins?); license NOW and relicense risk (the SSPL/BSL wave stranded real users).

4. **Price the exit before the entrance:** if wrong in 18 months, what does leaving cost? Adapter seams possible? Data/format portability (open formats cut exit cost 10×)? Migration paths others have walked? A mediocre tool with a cheap exit routinely beats a brilliant one with a moat — the most underweighted criterion, and vendors know it.

5. **Design the spike to probe YOUR risks, not the tutorial's path:** timeboxed (days), a vertical slice of your actual use case, deliberately hitting: your data's shape and scale, your deployment reality, your integration seams, and one failure drill (kill its backing store; read the error messages — the quality of a tool's failures is the quality of your future 3am). A spike that ends "it all worked great" probably probed nothing.

6. **Score against the step-1 criteria, decide with the score visible but not sovereign:** the weighted matrix structures the argument (`prioritization-frameworks` step 3's logic — surprises get discussed); the write-up is an ADR with drivers ranked, losers treated fairly, and the exit plan + revisit triggers as first-class consequences.

7. **Adopt on a leash:** start in ONE non-critical-but-real place, success criteria AND abandon-criteria named upfront ("if operational incidents exceed N in the first quarter, we execute the exit plan"), expand on evidence. The evaluation ends when the leash comes off — abandon-criteria written now are what make a wrong call cheap instead of endless.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "It has 30k stars — clearly healthy" | Stars measure past marketing; time-to-issue-response and contributor concentration measure present life. Plenty of lovingly-starred corpses on the awesome-lists. |
| "The tutorial went perfectly — it fits us" | The tutorial is marketing with syntax highlighting. The spike's job is finding where the candidate breaks against YOUR data shape, YOUR deployment, YOUR failure modes. |
| "We'll figure out migration if we ever need to leave" | Unpriced exits become 'we can't leave, we're too invested' as a permanent architecture principle. The exit cost is a criterion, not an afterthought — and it's the one vendors design against. |
| "The incumbent is boring — the new tool is 20% better" | Minus migration, minus retraining, minus unknown-unknowns, the 20% is usually negative. The status quo is a candidate and starts ahead; beating it needs more than novelty. |
| "The team's excited about it — that counts for something" | It counts as the criteria-contamination the step-1 ordering exists to prevent. Excitement post-criteria is a tiebreaker; excitement pre-criteria is résumé-driven selection. |
| "Four months of evaluation shows diligence" | Past the decision's actual stakes, evaluation cost exceeds the cost of being wrong. Timebox against stakes; the leash (step 7) is what makes a fast decision safe. |

## Red Flags

- Criteria document dated after the first demo.
- No incumbent row in the comparison.
- Vitals section citing stars and version numbers only.
- No exit-cost analysis anywhere.
- A spike report with zero findings.
- Adoption jumping straight to critical-path usage, no leash, no abandon-criteria.

## Verification

- [ ] Criteria + weights dated before candidate contact — doc linked.
- [ ] Shortlist eliminations logged one-line each.
- [ ] Vitals table complete per finalist (cadence, bus factor, hygiene, security, gravity, governance) — evidence linked.
- [ ] Exit priced per finalist — the paragraph exists.
- [ ] Spike report lists what BROKE and the failure-drill findings — attached.
- [ ] ADR written with ranked drivers and revisit triggers; leash scope + abandon-criteria recorded — links.

## Example

Need: workflow orchestration to replace cron-sprawl (40 jobs, growing dependencies — the requirement written from the incident pattern, pre-candidates). Shortlist: Airflow, Temporal, Dagster, + incumbent (cron + runbooks). Vitals: all three healthy on cadence; the differentiator was operational burden and the team fact that nobody had JVM/Go depth. Exit pricing: Dagster/Airflow DAG code moderately portable; Temporal's programming model deeply embedding (powerful AND a moat — noted honestly). Spikes, one week each, on the two finalists: real DAG (the gnarly month-end reconciliation chain), real volumes, and the failure drill — killed each one's DB mid-run; Dagster's error surfaced the exact failed step and resumed cleanly, Airflow's required log spelunking (the 3am-quality signal that outweighed two spreadsheet columns). Decision ADR: Dagster, drivers ranked (ops burden > ecosystem size), Airflow treated fairly, exit plan = DAGs as plain-Python assets. Leash: 5 jobs migrated, abandon-criteria named; leash off at month four. The revisit trigger (managed-Temporal pricing change) fired 18 months later and was evaluated in a day — because the ADR had pre-framed it.

## Related skills

- `architecture-decision-record` — the decision's written form.
- `dependency-vulnerability-audit` — the security half of the vitals.
- `prioritization-frameworks` — the scoring discipline shared.
- `vendor-evaluation` — the commercial layer for paid tools.
