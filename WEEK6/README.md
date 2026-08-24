# Snowflake Project Week — Cloud Data Platform at Scale

## Overview

This is a 5-day, project-based Snowflake week. Students learn every major Snowflake concept by
building one real data platform — no standalone theory lectures. By Friday, each student has a
working Snowflake setup with a role hierarchy, staged data loading at real volume, semi-structured
data handling with schema drift, automated pipelines (Streams & Task DAGs, Snowpipe), and
performance/cost controls tested against genuinely large data — and presents it.

**Structure:** 4 build days + 1 presentation/polish day.

| Day | Focus | Mode |
|-----|-------|------|
| Day 1 | Account setup + role hierarchy (RBAC) + Warehouses + Loading at scale (stages, COPY INTO, error handling) | Build |
| Day 2 | Semi-structured data at depth (nested JSON, schema drift) + Views + Time Travel + Cloning | Build |
| Day 3 | Automation: Streams + Task DAGs (multi-step CDC pipeline) + Snowpipe (auto-ingest) | Build |
| Day 4 | Performance & cost at scale: clustering, caching, concurrency, multi-cluster warehouses, resource monitors | Build |
| Day 5 | Polish, architecture walkthrough, presentation | Present |

---

## Project & dataset

The project is a Jaffle Shop style e-commerce platform, but built at real volume so the mechanics
of loading, automation, and performance tuning actually matter — not just execute successfully on
a handful of rows.

### Core dataset — generated at scale

Students generate their own data volume inside Snowflake using `GENERATOR`, `SEQ8()`, and
`UNIFORM()` rather than loading small static files:

- **`raw_customers`** — ~50K rows
- **`raw_orders`** — 1M+ rows, simulating 2+ years of order history with realistic skew (more
  orders near "now", weekday patterns)
- **`raw_payments`** — ~1.2M rows matched to orders, with some orders carrying multiple payment
  attempts (intentional messiness)
- **`raw_order_items.json`** — nested JSON with arrays of arrays (order → line items → product
  modifiers/add-ons), tens of thousands of documents, used for the VARIANT/FLATTEN day
- **`raw_orders_batch2.csv`** — a mid-week incoming batch of ~50K new orders, with a schema change
  (one new column, one renamed column) that forces students to handle schema drift in `COPY INTO`
  and in the Stream/Task pipeline

Generating the data themselves (rather than being handed static files) means students see exactly
how the volume and skew were built, and walk away with a reusable pattern for synthesizing test
data on any future project.

### Performance dataset — TPC-H sample data

Snowflake ships `SNOWFLAKE_SAMPLE_DATA.TPCH_SF10` and `SF100` as ready-to-query shares — tens to
hundreds of millions of rows, no loading required. This is used specifically on Day 4, where
clustering, caching, and warehouse-sizing lessons need data too large for a single micro-partition
to fake the result. Students run the same query against `TPCH_SF1`, `TPCH_SF10`, and `TPCH_SF100`
on different warehouse sizes and compare timings directly.

---

## Concepts covered

- Snowflake account structure: databases, schemas, warehouses
- Role hierarchy: `ACCOUNTADMIN` → `SYSADMIN` → functional custom roles (e.g. `DATA_ENGINEER`,
  `ANALYST_READONLY`), role inheritance, and future grants (`GRANT ... ON FUTURE TABLES IN
  SCHEMA ...`) so new objects inherit access automatically
- Internal stages, `PUT`, `COPY INTO` — including `ON_ERROR = 'CONTINUE'` vs `'SKIP_FILE'`,
  `VALIDATION_MODE`, and reading `COPY_HISTORY` / load errors
- File formats (CSV, JSON) for loading, including handling malformed rows in a batch
- Semi-structured data at depth: `VARIANT`, `:` and `[]` path notation, `FLATTEN()` with `LATERAL`
  joins across two nesting levels, and handling schema drift between batches
- Views vs. Materialized Views, and when a materialized view is a cost mistake rather than a win
- Time Travel (`AT`/`BEFORE`, `UNDROP`) — recovering from a bad `UPDATE`, not just a dropped object
- Zero-copy Cloning (`CREATE ... CLONE`) — cloning a multi-million-row table for a "dev"
  environment and confirming zero storage cost until divergence
- Streams (standard vs. `APPEND_ONLY`) and Task DAGs (a root task plus dependent child tasks)
- Snowpipe (continuous/auto-ingest loading), including checking `PIPE_STATUS` and load history
- Clustering keys and micro-partitions — chosen and justified using
  `SYSTEM$CLUSTERING_INFORMATION` on a real multi-million-row table
- Query result caching vs. warehouse (local disk) caching — demonstrated with cache disabled vs
  enabled
- Warehouse sizing, multi-cluster warehouses under simulated concurrent load, auto-suspend/
  auto-resume
- Resource monitors with multiple thresholds (notify at 50%, suspend at 100%)

---

## Repo / workspace structure

```
snowflake_platform_project/
├── README.md
├── sql/
│   ├── 01_setup/
│   │   ├── databases_schemas.sql
│   │   ├── role_hierarchy_and_grants.sql
│   │   └── warehouses.sql
│   ├── 02_loading/
│   │   ├── stage_setup.sql
│   │   ├── file_formats.sql
│   │   ├── generate_scaled_dataset.sql
│   │   └── copy_into_raw_tables.sql
│   ├── 03_semi_structured/
│   │   ├── load_order_items_json.sql
│   │   ├── flatten_nested_arrays.sql
│   │   └── handle_schema_drift.sql
│   ├── 04_views_time_travel_cloning/
│   │   ├── views.sql
│   │   ├── time_travel_demo.sql
│   │   └── clone_demo.sql
│   ├── 05_automation/
│   │   ├── streams.sql
│   │   ├── task_dag.sql
│   │   └── snowpipe_setup.sql
│   └── 06_performance_cost/
│       ├── clustering_keys.sql
│       ├── tpch_scale_comparison.sql
│       ├── concurrency_multicluster_test.sql
│       ├── query_profile_notes.md
│       └── resource_monitor.sql
├── data/
│   └── raw_orders_batch2.csv
└── assets/
    └── screenshots/
```

Pure Snowflake (SQL worksheets / Snowsight UI) — no dbt project this week.

---

## Daily deliverables

- **Day 1:** role hierarchy with at least 2 functional custom roles and future grants working;
  1M+ order rows generated and loaded via internal stage + `COPY INTO`; at least one intentionally
  malformed row handled with `ON_ERROR` and explained
- **Day 2:** nested JSON loaded and flattened across two array levels; one standard view and one
  materialized view created with a written note on when each is appropriate; Time Travel used to
  recover from a bad `UPDATE`; one zero-copy clone of the full orders table used as a "dev"
  environment
- **Day 3:** a Stream tracking changes on the orders table; a Task DAG (root task plus at least
  one dependent child) processing stream data into a summary table; the batch-2 load handled
  despite its schema drift; Snowpipe configured and demonstrated on the batch-2 file
- **Day 4:** a clustering key applied to a multi-million-row table and justified with
  `SYSTEM$CLUSTERING_INFORMATION`; caching behavior demonstrated with cold vs. cached run timings
  recorded; a concurrency test run against a multi-cluster warehouse compared to a single-cluster
  warehouse; a resource monitor with a notify threshold and a suspend threshold
- **Day 5:** polished README, architecture diagram/screenshots, 5–10 min presentation covering the
  platform, the scale decisions made, and what broke along the way (schema drift, load errors,
  concurrency limits)

---

## Day-wise files

Each day has two companion files:
- `dayN_concepts.md` — the concepts for that day, explained through the project
- `dayN_task.md` — the hands-on task/checklist to complete that day

Next: Day 1 concepts + task file, including the role hierarchy setup and the `GENERATOR`/`SEQ8`
script used to build the 1M+ row order dataset.