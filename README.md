# skillz

An open-source library of high-quality **Skills** for [Claude Code](https://claude.com/claude-code) — expert-written, repeatable workflows that teach Claude best practices for specific tasks, from debugging production systems to writing postmortems to pricing a SaaS product.

Each skill is a folder with a `SKILL.md`: YAML frontmatter tells Claude *when* to load it; the body is a step-by-step workflow with real pitfalls and worked examples, written as if an expert practitioner documented their own craft.

## Installing and using skills

Skills work with Claude Code's skill-loading mechanism. Options:

**Per-project:** copy the skill folders you want into your project's `.claude/skills/` directory:

```bash
mkdir -p .claude/skills
cp -r skillz/skills/testing-qa/flaky-test-diagnosis .claude/skills/
```

**Globally (all projects):** copy into `~/.claude/skills/`:

```bash
cp -r skillz/skills/sre-incident-response/incident-triage ~/.claude/skills/
```

Claude reads each skill's `description` to decide when it's relevant — you can also invoke one directly by asking for it ("use the flaky-test-diagnosis skill on this").

Grab individual skills, a category, or the whole set. Each skill is self-contained; cross-references to sibling skills are suggestions, not dependencies.

## Philosophy: what makes a good skill

A good skill is what a senior practitioner would tell you over your shoulder — not what a textbook chapter would say.

- **Triggered precisely.** The `description` states *when* to load it, with the phrases a user would actually type. Claude routes on descriptions alone; a vague one means the skill never fires or fires wrongly.
- **One job.** A skill does one thing. "Debug flaky tests" is a skill; "testing" is a category.
- **Executable, not aspirational.** Steps contain commands, thresholds, and decision criteria — "run `git bisect run ./check.sh`" beats "narrow down the failing commit."
- **Opinionated with escape hatches.** A default is recommended; the conditions to deviate are named. A list of five options without a pick is a survey, not a skill.
- **Honest about pitfalls.** The Common Pitfalls section carries real failure modes with their consequences, not generic warnings.
- **Proven by example.** Every skill ends with a worked scenario with real numbers.

A bad skill: restates documentation, hedges every recommendation, lists steps no one could execute ("ensure good quality"), or bundles three workflows behind one trigger.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full format spec.

## Skills index

**100 skills across 12 categories** (career-productivity and loop-engineering categories in progress).

### software-engineering

| Skill | Description |
|---|---|
| [code-review-checklist](skills/software-engineering/code-review-checklist/SKILL.md) | Systematic PR review ordered by severity: correctness, security, performance, style |
| [refactor-safely](skills/software-engineering/refactor-safely/SKILL.md) | Behavior-preserving restructuring with test gates between mechanical steps |
| [debugging-by-bisection](skills/software-engineering/debugging-by-bisection/SKILL.md) | Halve the search space: git bisect, input delta-debugging, config bisection |
| [legacy-code-first-contact](skills/software-engineering/legacy-code-first-contact/SKILL.md) | Characterization tests before touching unfamiliar, untested code |
| [api-design-rest](skills/software-engineering/api-design-rest/SKILL.md) | REST design: resources, status codes, pagination, idempotency, versioning |
| [error-handling-strategy](skills/software-engineering/error-handling-strategy/SKILL.md) | Throw vs return vs log; boundaries, loud fallbacks, swallowed-error audits |
| [dependency-upgrade](skills/software-engineering/dependency-upgrade/SKILL.md) | Safe version bumps: changelog reading, staged rollout, cheap rollback |
| [code-comments-that-last](skills/software-engineering/code-comments-that-last/SKILL.md) | Comment constraints and whys, not narration; kill comment rot |
| [monorepo-navigation](skills/software-engineering/monorepo-navigation/SKILL.md) | Find code, owners, and blast radius fast in large repos |

### testing-qa

| Skill | Description |
|---|---|
| [unit-test-design](skills/testing-qa/unit-test-design/SKILL.md) | Behavior-per-test, AAA structure, naming, and what not to test |
| [test-doubles-choice](skills/testing-qa/test-doubles-choice/SKILL.md) | Mock vs stub vs fake vs spy — decision tree plus overmocking fixes |
| [edge-case-enumeration](skills/testing-qa/edge-case-enumeration/SKILL.md) | Systematic input-space checklists: boundaries, nulls, Unicode, time, concurrency |
| [flaky-test-diagnosis](skills/testing-qa/flaky-test-diagnosis/SKILL.md) | Root-cause intermittent tests: interference, nondeterminism, CI deltas |
| [integration-test-strategy](skills/testing-qa/integration-test-strategy/SKILL.md) | Test the seams with real engines; testcontainers, contract tests, isolation |
| [property-based-testing](skills/testing-qa/property-based-testing/SKILL.md) | Generative tests from invariants: round-trips, oracles, shrinking |
| [regression-test-from-bug](skills/testing-qa/regression-test-from-bug/SKILL.md) | Failing test first, from every bug report, at the lowest exhibiting layer |
| [coverage-analysis](skills/testing-qa/coverage-analysis/SKILL.md) | Read coverage honestly: branch mode, risk-ranked gaps, mutation spot-checks |
| [e2e-test-triage](skills/testing-qa/e2e-test-triage/SKILL.md) | Shrink and stabilize browser suites; journey-critical only on the PR gate |

### devops-cicd

| Skill | Description |
|---|---|
| [ci-pipeline-design](skills/devops-cicd/ci-pipeline-design/SKILL.md) | Fail-fast stage ordering, caching, sharding, and trigger routing under a 10-min budget |
| [dockerfile-optimization](skills/devops-cicd/dockerfile-optimization/SKILL.md) | Layer ordering, multi-stage builds, cache mounts, image size and security floor |
| [github-actions-authoring](skills/devops-cicd/github-actions-authoring/SKILL.md) | Workflow triggers, permissions, SHA-pinning, injection safety, debugging |
| [deployment-rollback-plan](skills/devops-cicd/deployment-rollback-plan/SKILL.md) | Expand/contract migrations, N/N-1 compatibility, kill switches, tested rollbacks |
| [secrets-management](skills/devops-cicd/secrets-management/SKILL.md) | Where secrets live, OIDC over static keys, rotation as routine, commit blocking |
| [infra-as-code-review](skills/devops-cicd/infra-as-code-review/SKILL.md) | Read the plan not the diff: destroys, drift, blast radius, IAM wildcards |
| [environment-parity](skills/devops-cicd/environment-parity/SKILL.md) | Same artifact everywhere; classify divergences; close the data-shape gap |
| [release-versioning](skills/devops-cicd/release-versioning/SKILL.md) | Semver by consumer impact, automated bumps, tag protocol, changelog mechanics |

### sre-incident-response

| Skill | Description |
|---|---|
| [incident-triage](skills/sre-incident-response/incident-triage/SKILL.md) | First 15 minutes: severity, roles, mitigate-before-diagnose, timeline log |
| [production-debugging](skills/sre-incident-response/production-debugging/SKILL.md) | Metrics→logs→traces funnel, what-changed anchoring, falsifiable hypotheses |
| [blameless-postmortem](skills/sre-incident-response/blameless-postmortem/SKILL.md) | Contributing factors over root cause; action items that survive the backlog |
| [alerting-design](skills/sre-incident-response/alerting-design/SKILL.md) | Symptom-based paging, burn rates, three alert lanes, monthly noise review |
| [slo-definition](skills/sre-incident-response/slo-definition/SKILL.md) | Journey SLIs, targets from baselines, error budgets with pre-agreed policy |
| [runbook-authoring](skills/sre-incident-response/runbook-authoring/SKILL.md) | Verify→mitigate→diagnose→escalate docs a 3am responder can execute cold |
| [capacity-planning](skills/sre-incident-response/capacity-planning/SKILL.md) | Find the real constraint, load-test to the knee, headroom by reaction time |
| [on-call-handoff](skills/sre-incident-response/on-call-handoff/SKILL.md) | Structured rotation transfer: open items, smoldering risks, access checks |

### security

| Skill | Description |
|---|---|
| [threat-modeling](skills/security/threat-modeling/SKILL.md) | STRIDE walk over trust boundaries; ranked threats with explicit dispositions |
| [secure-code-review](skills/security/secure-code-review/SKILL.md) | Vulnerability-class review with grep patterns: authz, injection, secrets, SSRF |
| [dependency-vulnerability-audit](skills/security/dependency-vulnerability-audit/SKILL.md) | CVE triage by reachability and context, not CVSS headline numbers |
| [authn-authz-design](skills/security/authn-authz-design/SKILL.md) | Sessions vs JWT honestly, RBAC+ownership, deny-by-default, tenancy below the app |
| [input-validation-boundaries](skills/security/input-validation-boundaries/SKILL.md) | Parse-don't-validate at boundaries; allowlists; canonicalize-then-check |
| [secrets-incident-response](skills/security/secrets-incident-response/SKILL.md) | Leaked credential playbook: rotate first, audit usage, purge, fix the class |
| [security-headers-config](skills/security/security-headers-config/SKILL.md) | CSP via report-only, HSTS rollout order, CORS as allowlist, cookie armor |
| [least-privilege-review](skills/security/least-privilege-review/SKILL.md) | Granted-vs-used diffs, wildcard hunting, escalation graphs, JIT admin |

### data-analytics

| Skill | Description |
|---|---|
| [exploratory-data-analysis](skills/data-analytics/exploratory-data-analysis/SKILL.md) | Grain first, then nulls, distributions, time gaps, outliers — ending in written findings |
| [sql-query-optimization](skills/data-analytics/sql-query-optimization/SKILL.md) | EXPLAIN ANALYZE reading, composite indexes, sargability, keyset pagination |
| [data-cleaning-pipeline](skills/data-analytics/data-cleaning-pipeline/SKILL.md) | Scripted, staged cleaning with reject lanes, survivor rules, reconciliation |
| [metric-definition](skills/data-analytics/metric-definition/SKILL.md) | Pinned numerators/denominators, edge-case legislation, guardrail pairing |
| [ab-test-analysis](skills/data-analytics/ab-test-analysis/SKILL.md) | Pre-registration, no peeking, SRM checks, effect sizes over verdicts |
| [medallion-architecture-design](skills/data-analytics/medallion-architecture-design/SKILL.md) | Bronze/silver/gold contracts, promotion gates, rebuild-from-raw as a drill |
| [spark-etl-debugging](skills/data-analytics/spark-etl-debugging/SKILL.md) | Skew, shuffle spills, OOM, and stage retries — diagnosed from the Spark UI |
| [data-pipeline-idempotency](skills/data-analytics/data-pipeline-idempotency/SKILL.md) | Overwrite-partition and merge patterns, logical dates, watermarks, double-run tests |
| [cohort-retention-analysis](skills/data-analytics/cohort-retention-analysis/SKILL.md) | Cohort matrices read correctly: same-age columns, composition confounds |

### dsa-algorithms

| Skill | Description |
|---|---|
| [complexity-analysis](skills/dsa-algorithms/complexity-analysis/SKILL.md) | Derive real bounds: hidden costs, amortized analysis, the n-vs-budget table |
| [data-structure-selection](skills/dsa-algorithms/data-structure-selection/SKILL.md) | Choose by access pattern with the decision table and its real-world corrections |
| [two-pointer-sliding-window](skills/dsa-algorithms/two-pointer-sliding-window/SKILL.md) | Recognize and apply the patterns — with the monotonicity test that gates them |
| [graph-traversal-patterns](skills/dsa-algorithms/graph-traversal-patterns/SKILL.md) | BFS/DFS/toposort/union-find chosen by question; templates with invariants |
| [dynamic-programming-derivation](skills/dsa-algorithms/dynamic-programming-derivation/SKILL.md) | State sentences, recurrences from the last decision, memo-then-tabulate |
| [binary-search-variants](skills/dsa-algorithms/binary-search-variants/SKILL.md) | One boundary-finding template; search-on-answer; off-by-one proofing |
| [heap-and-priority-patterns](skills/dsa-algorithms/heap-and-priority-patterns/SKILL.md) | Top-k, k-way merge, scheduling, running medians, lazy deletion |
| [string-algorithm-toolkit](skills/dsa-algorithms/string-algorithm-toolkit/SKILL.md) | When naive is fine, and when Aho-Corasick, rolling hashes, or tries earn it |

### writing-docs

| Skill | Description |
|---|---|
| [readme-authoring](skills/writing-docs/readme-authoring/SKILL.md) | First-screen what/why/quickstart, tested examples, the fresh-clone test |
| [api-documentation](skills/writing-docs/api-documentation/SKILL.md) | Time-to-first-call walkthroughs, realistic examples, error catalogs, drift alarms |
| [architecture-decision-record](skills/writing-docs/architecture-decision-record/SKILL.md) | ADRs with fair options, both-direction consequences, ranked drivers |
| [technical-blog-post](skills/writing-docs/technical-blog-post/SKILL.md) | Takeaway-first structure, honest dead ends, evidence with axes |
| [changelog-writing](skills/writing-docs/changelog-writing/SKILL.md) | Reader-impact entries, loud breaking changes, curated not dumped |
| [onboarding-doc-design](skills/writing-docs/onboarding-doc-design/SKILL.md) | Timeline-structured day-1/week-1/month-1 docs with the new-hire audit flywheel |
| [docs-information-architecture](skills/writing-docs/docs-information-architecture/SKILL.md) | Diátaxis quadrants, journey-priority filling, goal-based navigation |
| [release-notes](skills/writing-docs/release-notes/SKILL.md) | Audience-framed announcements curated from the changelog; action-required first |
| [natural-prose-editing](skills/writing-docs/natural-prose-editing/SKILL.md) | Edit stiff or AI-sounding text into natural, specific prose (writing quality, not detector evasion) |

### product-pm

| Skill | Description |
|---|---|
| [prd-writing](skills/product-pm/prd-writing/SKILL.md) | Evidence-backed problem statements, non-goals, measurable success, open questions |
| [user-story-slicing](skills/product-pm/user-story-slicing/SKILL.md) | Vertical slices via the split catalog; INVEST; spikes for uncertainty |
| [prioritization-frameworks](skills/product-pm/prioritization-frameworks/SKILL.md) | RICE/ICE/Kano matched to decision type; scores structure argument, humans decide |
| [roadmap-communication](skills/product-pm/roadmap-communication/SKILL.md) | Now/next/later with honest certainty gradients; problems over dates |
| [stakeholder-update](skills/product-pm/stakeholder-update/SKILL.md) | One-screen progress/risks/asks on a metronome; bad news early |
| [feature-scoping-cut](skills/product-pm/feature-scoping-cut/SKILL.md) | Protect the kernel, cut along standard axes, disposition every cut |
| [user-interview-synthesis](skills/product-pm/user-interview-synthesis/SKILL.md) | Observation/interpretation layers, behavior over opinion, graded findings |
| [competitive-analysis](skills/product-pm/competitive-analysis/SKILL.md) | Jobs-based teardowns ending in compete/counter/concede/ignore calls |

### design-ux

| Skill | Description |
|---|---|
| [ui-heuristic-review](skills/design-ux/ui-heuristic-review/SKILL.md) | Task-first Nielsen pass with severity-rated, principle-anchored findings |
| [accessibility-audit](skills/design-ux/accessibility-audit/SKILL.md) | Keyboard walks, screen-reader passes, focus management — beyond the scanner |
| [design-system-tokens](skills/design-ux/design-system-tokens/SKILL.md) | Primitive/semantic layers, constrained scales, theming as re-binding |
| [form-design](skills/design-ux/form-design/SKILL.md) | Field deletion, validation timing, input-layer wins, per-field instrumentation |
| [empty-loading-error-states](skills/design-ux/empty-loading-error-states/SKILL.md) | The state matrix: empties by cause, duration-matched loading, exits from errors |
| [responsive-layout-strategy](skills/design-ux/responsive-layout-strategy/SKILL.md) | Intrinsic layout first, content breakpoints, container queries, awkward-width testing |
| [microcopy-writing](skills/design-ux/microcopy-writing/SKILL.md) | Buttons that say what they do, two-part errors, undo over confirms, term sheets |
| [wireframe-to-spec](skills/design-ux/wireframe-to-spec/SKILL.md) | State matrices, data rules, behavior contracts — the mock's missing frames |

### research

| Skill | Description |
|---|---|
| [literature-review](skills/research/literature-review/SKILL.md) | Ringed searches, two-pass triage, synthesis matrices, claims-based writeups |
| [technology-evaluation](skills/research/technology-evaluation/SKILL.md) | Maintenance vitals, exit pricing, spikes that probe your risks, adopt on a leash |
| [survey-design](skills/research/survey-design/SKILL.md) | Analysis-backward instruments, unleading wording, sampling honesty, pilots |
| [source-credibility-check](skills/research/source-credibility-check/SKILL.md) | Chase claims to primary sources; incentive audits; independent corroboration |
| [research-question-framing](skills/research/research-question-framing/SKILL.md) | Extract the decision, pin every noun, gate on answerability, fence the scope |
| [experiment-design](skills/research/experiment-design/SKILL.md) | Mechanism hypotheses, honest comparisons, confound hunting, pre-registration |
| [note-synthesis](skills/research/note-synthesis/SKILL.md) | Notes→claims→clusters→outline; contradictions as cargo |
| [citation-management](skills/research/citation-management/SKILL.md) | Capture at encounter, bind at writing, archive against link rot, audit before ship |

### finance-ops

| Skill | Description |
|---|---|
| [budget-variance-analysis](skills/finance-ops/budget-variance-analysis/SKILL.md) | Materiality gates, variance-nature triage, driver math, reforecast implications |
| [unit-economics-model](skills/finance-ops/unit-economics-model/SKILL.md) | Contribution margin, fully-loaded CAC, cohort-based LTV, payback as co-headline |
| [invoice-expense-workflow](skills/finance-ops/invoice-expense-workflow/SKILL.md) | Risk-tiered approvals, separation of duties, renewals calendars, audit trails |
| [financial-model-review](skills/finance-ops/financial-model-review/SKILL.md) | Hardcode hunts, structural integrity, assumption audits, sensitivity verdicts |
| [pricing-analysis](skills/finance-ops/pricing-analysis/SKILL.md) | Value-anchored pricing, WTP evidence, metric choice, tier fences, discount discipline |
| [cash-flow-forecast](skills/finance-ops/cash-flow-forecast/SKILL.md) | 13-week direct forecasts, behavioral collections, scenarios, trigger ladders |
| [vendor-evaluation](skills/finance-ops/vendor-evaluation/SKILL.md) | 3-year TCO, contract terms that bite, negotiation leverage, renewal calendars |
| [kpi-reporting-pack](skills/finance-ops/kpi-reporting-pack/SKILL.md) | Decision-driving KPI pages with trends, guardrails, and exception-based reviews |

## Contributing

New skills welcome — see [CONTRIBUTING.md](CONTRIBUTING.md) for the format spec and quality bar, and [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md). One skill per PR preferred.

## License

[MIT](LICENSE)
