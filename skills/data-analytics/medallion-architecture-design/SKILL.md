---
name: medallion-architecture-design
description: >
  Use when designing lakehouse/warehouse layers — bronze/silver/gold
  responsibilities, promotion rules between layers, schema and quality
  contracts per layer. Triggers: "medallion architecture", "bronze silver
  gold", "design our data layers", "lakehouse structure", "staging vs
  marts", "where should this transformation live". NOT for the mechanics
  of any single pipeline run (see data-pipeline-idempotency) — this is
  the map those pipelines live on.
---

# Medallion Architecture Design

## Overview

One job per layer — bronze preserves, silver conforms, gold serves — with promotion gates that block instead of advise, read paths enforced by grants, and rebuild-from-raw drilled before it's needed in anger. The architecture's superpower is reprocessing; everything else defends it.

## When to Use

- Structuring a lakehouse/warehouse (Databricks, Snowflake, BigQuery + dbt) from scratch or untangling one where every table reads from every other.
- Deciding which layer a given transformation belongs in.

**When NOT to use:**
- Per-run pipeline semantics — `data-pipeline-idempotency`.

## Prerequisites

- Inventory of sources (systems, formats, arrival patterns) and the consuming workloads (BI, ML, reverse-ETL).

## The Workflow

1. **Assign each layer ONE job, and write the contract down:**
   - **Bronze — preserve.** Raw, as-received, append-only, immutable. Schema = source schema plus lineage columns (`_ingested_at`, `_source_file`, `_batch_id`). No business logic, no dedupe, not even type fixes — bronze's value is being the incorruptible replay source. Bad data lands in bronze *by design*.
   - **Silver — conform.** Cleaned, typed, deduplicated, conformed entities at a defined grain: one table per business entity, source systems merged, quality rules enforced (`data-cleaning-pipeline` stages live here). Silver is the layer analysts *join*.
   - **Gold — serve.** Consumption-shaped: aggregates, wide marts, feature tables, metric materializations (`metric-definition`'s canonical implementations). Shaped per consumer; allowed to be opinionated and duplicated-by-design.

2. **Define promotion rules as executable gates, not vibes.** Bronze→silver: schema validation (failures → quarantine with reasons, never silently dropped — the reconciliation must hold: bronze = silver + quarantine, per batch), dedupe with survivorship, required-field and referential checks. Silver→gold: grain assertions, reconciliation totals, freshness SLAs. Encode as dbt tests / Delta constraints / expectations suites that *block promotion* on failure, or page the owner (`alerting-design` lanes apply).

3. **Make the layer boundaries the ONLY read paths.** Gold reads silver; silver reads bronze; consumers read gold (analysts may read silver; nobody queries bronze except reprocessing jobs). Enforce with warehouse grants (`least-privilege-review` energy), not policy docs — every "this dashboard reads bronze because deadline" becomes permanent load-bearing spaghetti.

4. **Handle the two classic modeling tensions explicitly:**
   - **Where does business logic live?** Logic that makes data *correct* (dedupe, type, unify enums) is silver; logic that makes data *useful for a purpose* (fiscal calendars, attribution rules, KPI formulas) is gold. Two consumers wanting different definitions = two gold tables, never a silver fork.
   - **Slowly changing dimensions:** decide per entity at silver — SCD2 for entities whose past states matter (customer plan tier for revenue restatement), SCD1 for correction-only attributes. Retrofitting SCD2 is painful; SCD2-everywhere triples storage and join complexity (and someone always forgets `WHERE _current`). Per-entity, in the design doc.

5. **Design for reprocessing from day one.** The superpower is "logic bug found → rebuild silver/gold from bronze." Requires: bronze partitioned for selective replay, silver/gold builds idempotent (`data-pipeline-idempotency`), and versioned transformation code so "which logic built this table?" is answerable. Test the superpower: rebuild one silver table from bronze in staging BEFORE you need it in anger.

6. **Attach ownership and metadata per layer:** bronze owned by ingestion/platform, silver by data engineering, gold by the consuming domain. Catalog entries (even a README per schema) stating grain, refresh cadence, contract, owner. Orphan tables are where trust goes to die.

7. **Keep the layer count honest.** Three is the default, not a law — a landing zone for CDC staging or a split gold can earn their place. What never earns its place: silver-2, silver-3, and gold-but-actually-silver tables breeding because step 3 wasn't enforced. If a table's layer can't be stated in one sentence, it's in the wrong layer or shouldn't exist.

## Common Rationalizations

| Excuse | Why it doesn't hold |
|---|---|
| "Filter the malformed rows at ingest — keep bronze clean" | A 'clean' bronze is a corrupted replay source: the rows you filter today are the evidence you need next quarter, and the bug you filter around deletes its own trace. Quarantine at the silver gate; bronze preserves. |
| "Put the KPI logic in silver — everyone needs it" | Until marketing and finance need different attribution, and someone forks silver. Correctness in silver, purpose in gold — the split exists because 'everyone' never stays one audience. |
| "The dashboard reads bronze just this once — deadline" | The once becomes load-bearing by next sprint, bypassing every quality gate, and its numbers get compared against gold's in an exec meeting. Grants, not promises (step 3). |
| "Row counts look close enough between layers" | 'Close' means rows are vanishing unaccounted. The per-batch reconciliation (bronze = silver + quarantine) is cheap arithmetic; without it, 'bronze says 1.2M, gold says 1.1M' takes a week to explain. |
| "SCD2 everywhere — history is always valuable" | Triple storage, every join needing `_current`, and someone always forgets — wrong numbers, silently. History per analytical need (step 4), not per ideology. |
| "We'll test the rebuild when we actually need it" | The first real reprocessing attempt is when you discover the non-idempotent gold job and the unpartitioned bronze. The staging drill (step 5) finds both at rehearsal prices. |

## Red Flags

- Dashboards/queries reading bronze directly.
- Bronze tables with business logic or dropped rows at ingest.
- Rows unaccounted between layers; no quarantine tables anywhere.
- The same entity conformed differently in two "silver" tables.
- SCD2 joins missing `_current` filters in consuming queries.
- No rebuild has ever been executed; nobody can state the replay procedure.
- Tables whose layer nobody can name in one sentence.

## Verification

- [ ] Layer contracts written (per-layer job, schema policy, lineage columns) — doc linked.
- [ ] Promotion gates executable and blocking — dbt test/constraint config linked; a deliberately bad batch shown quarantined with reconciliation exact.
- [ ] Read paths enforced by grants — permissions config shown; bronze unreadable to analyst roles.
- [ ] SCD decisions recorded per entity with their analytical justification.
- [ ] Rebuild drill completed: one silver table reconstructed from bronze in staging — run linked, duration noted.
- [ ] Ownership + catalog entries present for every schema — spot-check three.

## Example

Scale-up with a "modern data stack" that grew organically: 400 dbt models, dashboards reading raw Fivetran tables, three definitions of `orders`. Redesign: bronze = Fivetran/CDC landing, append-only with `_batch_id`, grants revoked from analysts; silver = 9 conformed entities (the three `orders` variants unified — one week of archaeology on which was right, findings written into the silver contract), SCD2 on `customers.plan` only (revenue restatement need), quarantine tables with per-batch reconciliation tests in dbt; gold = finance mart, product mart, ML feature tables, each owned by its domain team. Promotion gates as dbt tests blocking downstream builds. Rebuild drill in month 2: silver reconstructed from 90 days of bronze in 3 hours — which then got used for real in month 5 when a currency bug was found in silver logic; the fix + replay took an afternoon instead of a quarter of "numbers were wrong, we can't say by how much."

## Related skills

- `data-pipeline-idempotency` — the per-pipeline property step 5 requires.
- `data-cleaning-pipeline` — the bronze→silver transformation content.
- `metric-definition` — what gold materializes.
- `spark-etl-debugging` — when the promotion jobs themselves misbehave.
