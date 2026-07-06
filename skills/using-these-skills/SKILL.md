---
name: using-these-skills
description: >
  Load this at the start of any session using this library. Explains how
  these skills work (processes to follow, not docs to summarize), routes
  task types to the right skill across all 14 categories, and states
  the repo-wide non-negotiables. Triggers: session start, "which skill
  applies", "how do I use these skills", task routing. NOT a skill for
  any specific task — it's the map to the ones that are.
---

# Using These Skills

## Overview

These skills are workflows the agent follows, not reference documents to summarize. Each one is a numbered process with exit criteria; loading a skill means executing its steps and passing its Verification checklist — reading it and proceeding on vibes is the failure mode the format exists to prevent.

## Repo-wide non-negotiables

These apply whenever any skill from this library is active:

1. **Surface assumptions before implementing anything non-trivial.** State what you're assuming (inputs, environment, intent) where the user can see it — before the work, not in the apology after.
2. **Stop and ask rather than silently guessing on ambiguous requirements.** A wrong guess executed confidently costs more than the question. If the requirement forks and the fork matters, the user decides.
3. **Never skip a skill's Verification section.** The task isn't done until the checklist passes with evidence. "It looks right" is not a checklist state.
4. **Rationalization tables are binding.** If you catch yourself generating an excuse that appears in the active skill's Common Rationalizations table, that's the signal to follow the step, not skip it.

## Task routing

Find the row that matches the task's shape; load that skill (and its Related skills as needed).

```
What shape is the task?
│
├─ Writing/changing CODE
│   ├─ reviewing a change ............... code-review-checklist (security surface? secure-code-review)
│   ├─ restructuring, no behavior change  refactor-safely (untested code first? legacy-code-first-contact)
│   ├─ hunting an unknown cause ......... debugging-by-bisection (no repro, live system? production-debugging)
│   ├─ fixing a reported bug ............ regression-test-from-bug
│   ├─ designing an HTTP API ............ api-design-rest
│   ├─ failure paths / catches .......... error-handling-strategy (+ retry-and-backoff-strategy)
│   ├─ version bumps .................... dependency-upgrade (CVE-driven? dependency-vulnerability-audit)
│   └─ finding code/owners in a big repo  monorepo-navigation
│
├─ TESTING
│   ├─ writing unit tests ............... unit-test-design (+ edge-case-enumeration)
│   ├─ mocks/stubs/fakes decision ....... test-doubles-choice
│   ├─ intermittent failures ............ flaky-test-diagnosis
│   ├─ what to test with real parts ..... integration-test-strategy
│   ├─ invariants over big input spaces . property-based-testing
│   ├─ reading coverage ................. coverage-analysis
│   └─ slow/flaky browser suite ......... e2e-test-triage
│
├─ SHIPPING / INFRA
│   ├─ CI shape and speed ............... ci-pipeline-design (Actions syntax? github-actions-authoring)
│   ├─ container builds ................. dockerfile-optimization
│   ├─ deploys + migrations + rollback .. deployment-rollback-plan
│   ├─ credentials anywhere ............. secrets-management (already leaked? secrets-incident-response)
│   ├─ Terraform/IaC changes ............ infra-as-code-review
│   ├─ works-in-staging-not-prod ........ environment-parity
│   └─ versions/tags/changelogs ......... release-versioning
│
├─ PRODUCTION / OPERATIONS
│   ├─ active incident, users down ...... incident-triage  ← FIRST, before diagnosis
│   ├─ live diagnosis, no repro ......... production-debugging
│   ├─ after resolution ................. blameless-postmortem
│   ├─ pager noise / missing alerts ..... alerting-design (targets? slo-definition)
│   ├─ operational procedures ........... runbook-authoring
│   ├─ will-we-survive-the-load ......... capacity-planning
│   └─ rotation change .................. on-call-handoff
│
├─ SECURITY
│   ├─ design-time "what could go wrong"  threat-modeling
│   ├─ code-level vuln hunting .......... secure-code-review
│   ├─ scanner/CVE triage ............... dependency-vulnerability-audit
│   ├─ auth design ...................... authn-authz-design
│   ├─ untrusted input handling ......... input-validation-boundaries
│   ├─ leaked credential — NOW .......... secrets-incident-response
│   ├─ CSP/CORS/HSTS .................... security-headers-config
│   └─ permissions too broad ............ least-privilege-review
│
├─ DATA
│   ├─ new dataset, first contact ....... exploratory-data-analysis
│   ├─ slow SQL ......................... sql-query-optimization (Spark/distributed? spark-etl-debugging)
│   ├─ messy data → clean ............... data-cleaning-pipeline
│   ├─ defining a metric ................ metric-definition
│   ├─ A/B result reading ............... ab-test-analysis
│   ├─ warehouse layering ............... medallion-architecture-design
│   ├─ rerun-safe pipelines ............. data-pipeline-idempotency
│   └─ retention/churn questions ........ cohort-retention-analysis
│
├─ ALGORITHMS (incl. interview prep)
│   ├─ will it scale / big-O ............ complexity-analysis
│   ├─ which container .................. data-structure-selection
│   ├─ subarray/substring/pairs ......... two-pointer-sliding-window
│   ├─ reachability/ordering/groups ..... graph-traversal-patterns
│   ├─ counting/optimizing choices ...... dynamic-programming-derivation
│   ├─ boundary/answer search ........... binary-search-variants
│   ├─ repeated min/max, top-k .......... heap-and-priority-patterns
│   └─ string matching at scale ......... string-algorithm-toolkit
│
├─ WRITING / DOCS
│   ├─ README ........................... readme-authoring
│   ├─ API docs for consumers ........... api-documentation
│   ├─ recording a decision ............. architecture-decision-record
│   ├─ blog post / engineering article .. technical-blog-post
│   ├─ changelog vs announcement ........ changelog-writing / release-notes
│   ├─ new-hire docs .................... onboarding-doc-design
│   ├─ organizing a docs site ........... docs-information-architecture
│   └─ stiff/AI-sounding prose .......... natural-prose-editing
│
├─ PRODUCT / PM
│   ├─ requirements doc ................. prd-writing
│   ├─ breaking down an epic ............ user-story-slicing
│   ├─ ranking a backlog ................ prioritization-frameworks
│   ├─ roadmap outward .................. roadmap-communication
│   ├─ recurring status ................. stakeholder-update
│   ├─ won't fit the timeline ........... feature-scoping-cut
│   ├─ interview notes → findings ....... user-interview-synthesis
│   └─ competitor teardown .............. competitive-analysis
│
├─ DESIGN / UX
│   ├─ evaluating a UI .................. ui-heuristic-review
│   ├─ accessibility .................... accessibility-audit
│   ├─ design tokens / dark mode ........ design-system-tokens
│   ├─ forms ............................ form-design
│   ├─ empty/loading/error states ....... empty-loading-error-states
│   ├─ responsive layout ................ responsive-layout-strategy
│   ├─ UI copy .......................... microcopy-writing
│   └─ mock → buildable spec ............ wireframe-to-spec
│
├─ RESEARCH
│   ├─ what's known on a topic .......... literature-review
│   ├─ adopt a tool/library ............. technology-evaluation
│   ├─ questionnaire design ............. survey-design
│   ├─ can I trust this claim ........... source-credibility-check
│   ├─ vague ask → real question ........ research-question-framing
│   ├─ prove causation .................. experiment-design
│   ├─ notes → document ................. note-synthesis
│   └─ tracking sources ................. citation-management
│
├─ FINANCE / OPS
│   ├─ actual vs plan ................... budget-variance-analysis
│   ├─ CAC/LTV/payback .................. unit-economics-model
│   ├─ approval workflows ............... invoice-expense-workflow
│   ├─ auditing a spreadsheet model ..... financial-model-review
│   ├─ what to charge ................... pricing-analysis
│   ├─ runway / 13-week cash ............ cash-flow-forecast
│   ├─ vendor selection/renewal ......... vendor-evaluation
│   └─ leadership metrics pack .......... kpi-reporting-pack
│
├─ CAREER / PRODUCTIVITY
│   ├─ resume → posting ................. resume-tailoring
│   ├─ interview prep ................... interview-prep-technical
│   ├─ offer/raise negotiation .......... salary-negotiation
│   ├─ personal task system ............. weekly-review
│   ├─ meeting capture .................. meeting-notes-actions
│   ├─ focus time ....................... deep-work-scheduling
│   └─ promotion case ................... promotion-packet
│
└─ AGENT LOOPS (building/debugging agentic systems)
    ├─ structuring the loop ............. agent-loop-design
    ├─ stuck/repeating/thrashing ........ tool-use-loop-debugging
    ├─ retries and backoff .............. retry-and-backoff-strategy
    ├─ observing loop behavior .......... feedback-loop-instrumentation
    ├─ where humans approve ............. human-in-the-loop-checkpoints
    ├─ when to stop ..................... convergence-criteria-design
    ├─ measuring agent quality .......... eval-loop-for-agents
    ├─ states vs freeform ............... state-machine-vs-freeform-loop
    ├─ context budget per iteration ..... context-window-loop-management
    └─ hard cost limits ................. runaway-cost-guardrails
```

Multiple rows match → load the most specific one first; its Related skills section chains to the others. No row matches → say so rather than force-fitting the nearest skill.

## How activation works

Skills load based on task match against each skill's frontmatter `description` — the descriptions carry positive triggers AND explicit "NOT for" exclusions. When a task matches a description, follow that skill's Workflow and pass its Verification before declaring done.

## Related skills

All 117 of them — this is the map.
