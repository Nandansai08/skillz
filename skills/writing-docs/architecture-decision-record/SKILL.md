---
name: architecture-decision-record
description: >
  Use when recording a significant technical decision — writing an ADR with
  context, options considered, consequences, and status, so future readers
  know why, not just what. Triggers: "write an ADR", "document this decision",
  "why did we choose X" (when the answer should be written down), "record
  the trade-offs", "decision record for".
---

# Architecture Decision Record

## When to use this skill
- A decision is being made that will be expensive to reverse or that future engineers will question: technology choices, service boundaries, data models, auth approaches, build-vs-buy.
- Excavating a past undocumented decision ("retroactive ADR") before changing it.
- NOT for decisions with one reasonable option or trivially reversible ones — ADR-ing everything buries the ten that matter under two hundred that don't.

## Prerequisites
- The decision genuinely at or near its decision point (ADRs written months after the fact drift into justification essays — mark late ones as retroactive honestly).
- The template: [templates/adr.md](templates/adr.md).

## Workflow

1. **Write the context as forces, without foreshadowing the winner.** What problem, what constraints (scale numbers, team skills, deadlines, compliance, existing systems), what requirements distinguish the options. The test: a reader should finish the context section unable to guess which option won. Context written as a runway for the chosen option ("we needed something Kafka-shaped...") is advocacy, not record.

2. **Give every serious option a fair paragraph — including "do nothing."** Per option: what it is, its genuine strengths *in this context*, its costs/risks, and why it lost (or won). The strawman tell: an option described in one dismissive line. If an option wasn't seriously considered, don't list it; if it was, respect it — the future reader evaluating a migration needs the real trade-off landscape, and the losing option's strengths are exactly what they'll rediscover.

3. **State the decision in one unambiguous sentence, with its scope.** "We will use Postgres logical replication for cross-region reads, for the reporting workload only." Scope boundaries ("this does not cover the transactional path") prevent the ADR from being cited beyond its jurisdiction later.

4. **Write consequences in both directions — this is the section that earns the document.** What becomes easier; what becomes harder ("all consumers must now handle eventual consistency up to 30s"); new obligations created (that replication lag now needs monitoring — `alerting-design` work item); what would trigger revisiting ("if lag exceeds 5 min at 10× current write volume, this design is invalidated"). An ADR with only positive consequences is a sales doc; the negative consequences are the ones the future reader is hunting for.

5. **Record the decision drivers ranked — why THIS trade-off.** Two honest sentences: "Operational familiarity outweighed the throughput ceiling: the team runs Postgres today, and projected volume stays 100× below the ceiling for 3 years." That ranking (familiarity > throughput, here) is the actual decision; the same options with different driver-rankings produce different winners, and future readers with different drivers can re-derive legitimately.

6. **Keep the lifecycle honest:** status field (Proposed → Accepted → Deprecated/Superseded-by-ADR-NNN), immutable once accepted (new information → new ADR that supersedes, never silent edits — the historical record of "what we believed then" is the value), numbered sequentially, stored in-repo next to the code they govern (`docs/adr/NNN-title.md`) so they ride along in clones and greps.

7. **Review it like code, briefly:** the people who argued for losing options confirm their option was represented fairly (the best de-strawmanning mechanism available), and someone outside the debate reads for comprehensibility. Then link it: from the PR implementing it, from the runbook of the system it shaped, from the next ADR that touches the area.

## Common pitfalls
- ADR written as post-hoc justification — context reverse-engineered from the conclusion. Readers can smell it, and it torpedoes trust in the whole ADR log.
- Options section with one real option and two strawmen ("we could also write our own database, but..."). If it was never on the table, it doesn't belong.
- Consequence-free records: "Decision: microservices. Consequences: better scalability." The obligations, the newly-hard things, and the invalidation triggers are the payload (step 4).
- Editing an accepted ADR when circumstances change — silently rewriting history. Supersede, don't revise; the chain of ADRs IS the architecture's biography.
- ADRs in a wiki nobody visits, decoupled from the code — undiscoverable at exactly the moment someone's about to violate one. In-repo, linked from PRs.
- Waiting for consensus perfection before writing: the ADR is the *instrument* of the decision meeting, not its trophy — draft it as the proposal, argue in review, accept with the argument recorded.

## Example
Decision: event bus for order events — Kafka vs SQS+SNS vs Postgres outbox table. ADR drafted as the proposal *before* the decision meeting. Context: 40 events/sec today (~500/sec 3-year projection), 4-person platform team with zero Kafka operational experience, one consumer today (3 projected), exactly-once not required (consumers idempotent per `data-pipeline-idempotency`). Options: each got its fair paragraph — Kafka's replay/ordering strengths acknowledged AND its ops burden for a 4-person team; the outbox's simplicity AND its polling latency; SQS's managed-ness AND its ordering limits. Decision: outbox + SQS fan-out, scoped to order events only. Drivers ranked: ops burden > replay capability at current scale. Consequences included the negative ("no replay — if a consumer needs event history, that's a new ADR") and the tripwire ("revisit at sustained 200 events/sec"). Eighteen months later a new hire proposed Kafka in week two — was pointed at ADR-014, read the driver ranking, checked current volume (55/sec), and withdrew the proposal in one comment: the document paid for itself in one avoided re-litigation.

## Related skills
- `technology-evaluation` — the evaluation work that feeds the options section.
- `api-design-rest` — a source of decisions that commonly deserve ADRs.
- `code-comments-that-last` — the micro-scale version of "record why, not what."
- `stakeholder-update` — communicating the decision outward once recorded.
