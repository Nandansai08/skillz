---
name: architecture-decision-record
description: >
  Use when recording a significant technical decision — writing an ADR
  with context, options considered, consequences, and status, so future
  readers know why, not just what. Triggers: "write an ADR", "document
  this decision", "why did we choose X" (when the answer should be
  written down), "record the trade-offs", "decision record for". NOT for
  decisions with one reasonable option or trivially reversible ones —
  ADR-ing everything buries the ten that matter.
---

# Architecture Decision Record

## Overview

An ADR's value is the ranked drivers and the negative consequences — the parts a future reader needs to re-evaluate the decision when circumstances change. Context written as a runway for the winner, and options listed as strawmen, produce a sales doc with a version number.

## When to Use

- A decision expensive to reverse or certain to be questioned: technology choices, service boundaries, data models, auth approaches, build-vs-buy.
- Excavating a past undocumented decision (retroactive ADR — marked as such) before changing it.

**When NOT to use:**
- One-reasonable-option or trivially reversible decisions — volume buries signal.

## Prerequisites

- The decision genuinely at or near its decision point.
- The template: [templates/adr.md](templates/adr.md).

## The Workflow

1. **Write the context as forces, without foreshadowing the winner.** Problem, constraints (scale numbers, team skills, deadlines, compliance), and the requirements that distinguish options. The test: a reader finishing the context section cannot guess which option won. Context written as a runway ("we needed something Kafka-shaped...") is advocacy, not record.

2. **Give every serious option a fair paragraph — including "do nothing."** Per option: what it is, its genuine strengths *in this context*, its costs/risks, and why it lost (or won). The strawman tell: an option described in one dismissive line. If it wasn't seriously considered, don't list it; if it was, respect it — the losing option's strengths are exactly what a future migration evaluator needs.

3. **State the decision in one unambiguous sentence, with its scope.** "We will use Postgres logical replication for cross-region reads, for the reporting workload only." Scope boundaries prevent the ADR being cited beyond its jurisdiction later.

4. **Write consequences in both directions — the section that earns the document.** What becomes easier; what becomes harder ("all consumers must now handle eventual consistency up to 30s"); new obligations created (the replication lag now needs monitoring — ticket it); and revisit triggers ("if lag exceeds 5 min at 10× write volume, this design is invalidated"). An ADR with only positive consequences is a sales doc.

5. **Record the decision drivers ranked — why THIS trade-off.** Two honest sentences: "Operational familiarity outweighed the throughput ceiling: the team runs Postgres today, and projected volume stays 100× below the ceiling for 3 years." The ranking IS the decision; future readers with different drivers can re-derive legitimately.

6. **Keep the lifecycle honest:** status field (Proposed → Accepted → Deprecated/Superseded-by-ADR-NNN), immutable once accepted (new information → a superseding ADR, never silent edits — "what we believed then" is the value), numbered sequentially, stored in-repo next to the code they govern (`docs/adr/NNN-title.md`).

7. **Review it like code, briefly, then link it:** the people who argued for losing options confirm fair representation (the best de-strawmanning mechanism available); someone outside the debate reads for comprehensibility. Link from the implementing PR, the affected runbook, and the next ADR touching the area. Draft the ADR as the decision meeting's *instrument*, not its trophy — argue in review, accept with the argument recorded.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "We all agreed — writing it up is bureaucracy" | Eighteen months later a new hire re-litigates it from scratch, or worse, quietly reverses it. The example's ADR ended that conversation in one comment; the meeting it summarized would have been re-held annually. |
| "I'll write the context after we decide — cleaner" | Post-decision context gets reverse-engineered from the conclusion, and readers can smell it. Draft-as-proposal (step 7) gets the honest version and the meeting agenda in one artifact. |
| "Adding weak alternatives shows due diligence" | 'We could write our own database, but...' is a strawman that discredits the real comparison. List what was seriously considered; respect it or omit it. |
| "Consequences: better scalability. Done" | The future reader is hunting the NEGATIVE consequences and the revisit triggers — the parts that tell them when this decision expires. Positive-only ADRs answer no question anyone will ever ask. |
| "Circumstances changed — I'll just update the old ADR" | Silent edits rewrite history and break every reference to it. Supersede with a new number; the chain IS the architecture's biography. |
| "ADR everything — consistency" | Two hundred trivial ADRs bury the ten that matter, and the practice dies of its own weight. The expensive-to-reverse filter is the practice's survival mechanism. |

## Red Flags

- Context sections from which the winner is guessable in one paragraph.
- Options with one-line dismissals next to the winner's essay.
- No revisit triggers, no negative consequences anywhere.
- An accepted ADR with edit history after acceptance.
- ADRs in a wiki nobody links from PRs or code.
- The same decision argued twice in six months with no ADR cited either time.

## Verification

- [ ] Context passes the can't-guess-the-winner test — reviewed by someone outside the debate.
- [ ] Every listed option has strengths AND costs in this context; losing-option advocates signed off on their representation.
- [ ] Decision sentence includes explicit scope ("does not cover...").
- [ ] Consequences include ≥1 harder-now item, ≥1 new obligation (ticketed), and ≥1 measurable revisit trigger.
- [ ] Drivers ranked in writing.
- [ ] Stored in-repo, numbered, linked from the implementing PR — links shown.

## Example

Decision: event bus for order events — Kafka vs SQS+SNS vs Postgres outbox. ADR drafted as the proposal *before* the decision meeting. Context: 40 events/sec today (~500/sec 3-year projection), 4-person platform team with zero Kafka operational experience, exactly-once not required (consumers idempotent). Options: each got its fair paragraph — Kafka's replay/ordering strengths acknowledged AND its ops burden for a 4-person team; the outbox's simplicity AND its polling latency. Decision: outbox + SQS fan-out, scoped to order events only. Drivers ranked: ops burden > replay capability at current scale. Consequences included the negative ("no replay — if a consumer needs event history, that's a new ADR") and the tripwire ("revisit at sustained 200 events/sec"). Eighteen months later a new hire proposed Kafka in week two — was pointed at ADR-014, read the driver ranking, checked current volume (55/sec), and withdrew the proposal in one comment: the document paid for itself in one avoided re-litigation.

## Related skills

- `technology-evaluation` — the evaluation work that feeds the options section.
- `api-design-rest` — a source of decisions that commonly deserve ADRs.
- `code-comments-that-last` — the micro-scale version of "record why, not what."
- `stakeholder-update` — communicating the decision outward once recorded.
