---
name: medallion-architecture-design
description: >
  Use when designing lakehouse/warehouse layers — bronze/silver/gold
  responsibilities, promotion rules between layers, schema and quality
  contracts per layer. Triggers: "medallion architecture", "bronze silver
  gold", "design our data layers", "lakehouse structure", "staging vs marts",
  "where should this transformation live".
---

# Medallion Architecture Design

## When to use this skill
- Structuring a lakehouse/warehouse (Databricks, Snowflake, BigQuery + dbt) from scratch or untangling one where every table reads from every other.
- Deciding which layer a given transformation belongs in.
- NOT for the mechanics of any single pipeline run — that's `data-pipeline-idempotency`; this is the map those pipelines live on.

## Prerequisites
- Inventory of sources (systems, formats, arrival patterns) and the consuming workloads (BI, ML, reverse-ETL).

## Workflow

1. **Assign each layer ONE job, and write the contract down:**
   - **Bronze — preserve.** Raw, as-received, append-only, immutable. Schema = source schema plus lineage columns (`_ingested_at`, `_source_file`, `_batch_id`). No business logic, no dedupe, not even type fixes — bronze's value is being the incorruptible replay source. Bad data lands in bronze *by design*; that's what makes reprocessing possible.
   - **Silver — conform.** Cleaned, typed, deduplicated, conformed entities at a defined grain: one table per business entity (customers, orders), source systems merged, quality rules enforced. This is where `data-cleaning-pipeline` stages live. Silver is the layer analysts *join*.
   - **Gold — serve.** Consumption-shaped: aggregates, wide denormalized marts, feature tables, metric materializations (`metric-definition`'s canonical implementations). Shaped per consumer workload; allowed to be opinionated and duplicated-by-design.

2. **Define promotion rules as executable gates, not vibes.** Bronze→silver: schema validation (types coerce or row → quarantine), dedupe with survivorship rule, required-field null checks, referential checks — failures go to a quarantine table with reasons, never silently dropped (the reconciliation must hold: bronze = silver + quarantine, per batch). Silver→gold: grain assertions, reconciliation totals against silver, freshness SLAs. Encode as dbt tests / Delta constraints / Great Expectations suites that *block promotion* on failure, or at minimum page the owner (`alerting-design` lanes apply).

3. **Make the layer boundaries the ONLY read paths.** Gold reads silver; silver reads bronze; consumers read gold (analysts may read silver; nobody queries bronze except reprocessing jobs). Every violation ("this dashboard reads a bronze table because deadline") becomes permanent load-bearing spaghetti — enforce with warehouse grants (`least-privilege-review` energy), not policy docs.

4. **Handle the two classic modeling tensions explicitly:**
   - **Where does business logic live?** Rule of thumb: logic that makes data *correct* (dedupe, type, unify enums) is silver; logic that makes data *useful for a purpose* (fiscal calendars, attribution rules, KPI formulas) is gold. When two consumers want different definitions, that's two gold tables, not a silver fork.
   - **Slowly changing dimensions:** decide per entity at silver — SCD2 (history tracked) for entities whose past states matter (customer plan tier for revenue restatement), SCD1 (overwrite) for correction-only attributes. Retrofitting SCD2 later is painful; choosing it everywhere triples storage and join complexity. Choose per analytical need, in the design doc.

5. **Design for reprocessing from day one.** The architecture's superpower is "logic bug found → rebuild silver/gold from bronze." That requires: bronze partitioned for selective replay (by ingest date/source), silver/gold builds idempotent (`data-pipeline-idempotency` — deterministic outputs, overwrite-partition semantics), and versioned transformation code so you can answer "which logic built this table?" Test the superpower: rebuild one silver table from bronze in staging before you need it in anger.

6. **Attach ownership and metadata per layer:** bronze owned by ingestion/platform, silver by data engineering with source-system SME input, gold by the consuming domain (analytics engineers embedded with the business team). Catalog entries (even a README per schema) stating grain, refresh cadence, contract, owner. Orphan tables are where trust goes to die.

7. **Keep the layer count honest.** Three layers is the default, not a law — a "pre-bronze" landing zone for CDC merge staging, or a split gold (core marts vs experimental sandboxes) can earn their place. What never earns its place: silver-2, silver-3, and gold-but-actually-silver tables breeding because nobody enforced step 3. If a table's layer can't be stated in one sentence, it's in the wrong layer or shouldn't exist.

## Common pitfalls
- "Fixing" data in bronze (dropping malformed rows at ingest). Now the replay source lies, and the bug you filter today deletes the evidence you need next quarter. Bronze preserves; quarantine happens at the silver gate.
- Business KPI logic buried in silver "because everyone needs it" — until marketing and finance need different attribution and someone forks silver. Correctness in silver, purpose in gold.
- Dashboards reading bronze directly (step 3's violation) — the resulting numbers bypass every quality gate, then get compared against gold in an executive meeting.
- No quarantine reconciliation: rows silently vanish between layers and "bronze says 1.2M, gold says 1.1M" takes a week to explain. The per-batch accounting is cheap; build it in.
- SCD2 nowhere (history questions unanswerable) or SCD2 everywhere (every join needs `WHERE _current = true` and someone always forgets — wrong numbers, silently). Per-entity decision, documented.
- Designing layers without testing the rebuild. The first real reprocessing attempt discovers the non-idempotent gold job and the bronze partition scheme that requires scanning two years.

## Example
Scale-up with a "modern data stack" that grew organically: 400 dbt models, dashboards reading raw Fivetran tables, three definitions of `orders`. Redesign: bronze = Fivetran/CDC landing, append-only with `_batch_id`, grants revoked from analysts; silver = 9 conformed entities (the three `orders` variants unified — one week of archaeology on which was right, findings written into the silver contract), SCD2 on `customers.plan` only (revenue restatement need), quarantine tables with per-batch reconciliation tests in dbt; gold = finance mart, product mart, ML feature tables, each owned by its domain team. Promotion gates as dbt tests blocking downstream builds. Rebuild drill in month 2: silver reconstructed from 90 days of bronze in 3 hours — which then got used for real in month 5 when a currency bug was found in silver logic; the fix + replay took an afternoon instead of a quarter of "numbers were wrong, we can't say by how much."

## Related skills
- `data-pipeline-idempotency` — the per-pipeline property step 5 requires.
- `data-cleaning-pipeline` — the bronze→silver transformation content.
- `metric-definition` — what gold materializes.
- `spark-etl-debugging` — when the promotion jobs themselves misbehave.
