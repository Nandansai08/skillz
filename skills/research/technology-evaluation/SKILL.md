---
name: technology-evaluation
description: >
  Use when evaluating a library, framework, or tool for adoption —
  maintenance signals, exit costs, spike design, comparing against the
  incumbent honestly. Triggers: "should we use X", "evaluate this library",
  "X vs Y for our stack", "is this project maintained", "build vs buy vs
  adopt", "picking a framework".
---

# Technology Evaluation

## When to use this skill
- Choosing a dependency, platform, or tool with meaningful switching costs.
- Re-evaluating an incumbent that's aging badly (chronic CVEs, abandoned — `dependency-vulnerability-audit` step 7's referrals land here).
- NOT for surveying a research field (`literature-review`) — this ends in an adopt/reject decision on specific candidates.

## Prerequisites
- The requirement driving the search, stated independently of any candidate ("we need X-shaped capability under Y constraints" — a requirement written after falling for a tool is a justification).
- The incumbent/status-quo named as a candidate: "keep doing what we do" competes on equal footing.

## Workflow

1. **Write the evaluation criteria and their weights BEFORE touching candidates:** must-haves (disqualifiers — license compatibility, platform support, compliance), weighted wants (performance envelope, learning curve for THIS team, operational burden), and the context facts that bend everything (team's existing expertise, scale actually needed — not aspirational scale, deployment constraints). Criteria written after the demo inherit the demo's charisma.

2. **Shortlist to 2–3 + incumbent, quickly and defensibly:** the long list dies on must-haves and coarse signals in an afternoon — the evaluation's depth budget belongs to the finalists, not to a 9-column spreadsheet of everything on the awesome-list. Log the one-line eliminations (the "why not Z?" question WILL arrive in review — `architecture-decision-record` step 2's fairness applies).

3. **Read the maintenance vitals — the health check that predicts your next three years:**
   - **Cadence & bus factor:** commit/release recency AND contributor concentration (`git shortlog -sn` shape — one person = one burnout from abandonment; check if that person is paid to work on it).
   - **Issue/PR hygiene:** median time-to-response on issues (not count — popular projects have many; dead ones have many *unanswered*), whether maintainers engage with hard bugs or only easy PRs.
   - **Security posture:** CVE history handled how fast, security policy exists, releases signed.
   - **Ecosystem gravity:** who else depends on it (big dependents = shared fate = de facto support), Stack Overflow/Discord answer density for the errors you'll hit.
   - **Governance & money:** foundation-backed / company-backed (whose roadmap wins when they conflict with yours?) / hobby. License NOW and license-change risk (the relicense wave is recent history — SSPL/BSL moves stranded real users).

4. **Price the exit before the entrance:** if this choice is wrong in 18 months, what does leaving cost? Abstraction seams possible (the adapter that keeps the dependency at arm's length — `test-doubles-choice` step 3's wrapping)? Data/format portability (open formats and standard protocols cut exit cost 10×)? Migration paths others have walked? A mediocre tool with a cheap exit routinely beats a brilliant one with a moat — the evaluation's most underweighted criterion, and vendors know it.

5. **Design the spike to probe YOUR risks, not the tutorial's path:** timeboxed (days), building a vertical slice of your actual use case, deliberately hitting: your data's shape and scale (the 10M-row table, the weird legacy encoding), your deployment reality (the airgapped env, the old kernel), your integration seams, and one failure drill (kill its backing store; read the error messages — the quality of a tool's failures is the quality of your future 3am). The tutorial path is marketing; the spike's job is finding where the candidate breaks against your specifics, so a spike that ends "it all worked great" probably probed nothing.

6. **Score against the step-1 criteria, then decide with the score visible but not sovereign:** the weighted matrix structures the argument (`prioritization-frameworks` step 3's same logic — surprises get discussed, scores don't auto-decide); the write-up is an ADR (`architecture-decision-record`) with the drivers ranked, the losers treated fairly, and the exit plan + revisit triggers as first-class consequences.

7. **Adopt on a leash:** start in ONE non-critical-but-real place, with the success criteria and the abandon-criteria both named upfront ("if operational incidents from it exceed N in the first quarter, we execute the exit plan"), expand on evidence. The evaluation isn't over at the decision — it's over when the leash comes off, and the abandon-criteria written now are what make a wrong call cheap instead of endless (`runaway-cost-guardrails` energy, applied to adoption).

## Common pitfalls
- Résumé-driven and conference-driven selection: the tool chosen because it's exciting, criteria back-filled to fit. Step 1's ordering is the whole defense.
- GitHub-stars-as-health: stars measure past marketing; time-to-issue-response and contributor concentration measure present life (step 3). Plenty of 30k-star projects are lovingly-starred corpses.
- Tutorial-benchmarking: evaluating on the golden path with toy data, discovering the real-data cliff in production month two (step 5's your-risks spike).
- Ignoring the incumbent's home advantage: the challenger's 20% improvement, minus migration cost, minus the team's retraining, minus the unknown-unknowns discount, is usually negative — the status quo is a candidate, and it starts ahead.
- Exit-blindness: adopting the proprietary format/protocol/platform with no priced exit, then "we can't leave, we're too invested" as a permanent architecture principle (step 4).
- The eternal evaluation: month four, seventeen criteria, no decision — evaluation cost has exceeded the cost of being wrong. Timebox the whole thing against the decision's actual stakes.

## Example
Need: workflow orchestration to replace cron-sprawl (40 jobs, growing dependencies between them — the requirement written from the incident pattern, pre-candidates). Shortlist: Airflow, Temporal, Dagster, + incumbent (cron + runbooks). Vitals pass: all three healthy on cadence; the differentiator was operational burden (Airflow's scheduler ops vs managed options) and the team fact that nobody had JVM/Go depth (criteria weights, step 1). Exit pricing: Dagster/Airflow DAG code moderately portable; Temporal's programming model deeply embedding (powerful AND a moat — noted honestly). Spikes, one week each, against the two finalists: real DAG (the gnarly month-end reconciliation chain), real data volumes, and the failure drill — killed each one's DB mid-run; Dagster's error surfaced the exact failed step and resumed cleanly, Airflow's required log spelunking (the 3am-quality signal that outweighed two spreadsheet columns). Decision ADR: Dagster, drivers ranked (ops burden > ecosystem size), Airflow treated fairly, exit plan = DAGs kept in plain-Python assets. Leash: 5 jobs migrated, abandon-criteria named; leash off at month four, full migration by two quarters. The revisit trigger (managed-Temporal pricing change) fired 18 months later and was evaluated in a day — because the ADR had pre-framed it.

## Related skills
- `architecture-decision-record` — the decision's written form.
- `dependency-vulnerability-audit` — the security half of the vitals.
- `prioritization-frameworks` — the scoring discipline shared.
- `dependency-upgrade` — living with the choice's version treadmill.
