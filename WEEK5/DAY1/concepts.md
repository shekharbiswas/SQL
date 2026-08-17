# Day 1 — Concepts 

Today's build: connect dbt to Snowflake, load raw data, declare sources, and write your first staging models. Every concept below is taught *through* these steps — read the "why" before each corresponding task.

---

## 1. Project setup — `dbt init`

`dbt init` scaffolds a new dbt project: folders for `models/`, `seeds/`, `snapshots/`, `macros/`, `tests/`, plus a `dbt_project.yml` config file. This is the skeleton every dbt project follows — same structure whether it's a student project or a company with 2000 models.

## 2. `profiles.yml` — connecting to Snowflake

dbt doesn't store warehouse credentials inside the project (so it's safe to commit to Git). Instead, `profiles.yml` lives outside the repo (usually `~/.dbt/profiles.yml`) and holds:
- account, user, password
- role, warehouse, database, schema

This file is *how* dbt knows which Snowflake account/warehouse to run SQL against. One project can have multiple profiles (dev, prod) — today you'll only need `dev`.

## 3. Seeds — `dbt seed`

Seeds are static CSV files dbt loads directly into your warehouse as tables. They're meant for small, rarely-changing reference data (not your main raw data pipeline in real companies — but perfect for a training project since it avoids setting up ingestion tools).

Today you'll seed the Jaffle Shop raw CSVs (`raw_customers`, `raw_orders`, `raw_payments`) plus the synthetic `raw_product_prices_v2.csv` for later use.

## 4. Sources — `sources.yml` and `source()`

Once raw tables exist in Snowflake, dbt needs to know about them. A `sources.yml` file declares: "these tables exist, here's their schema/database, here's what to call them." Then in your SQL models, instead of hardcoding a table name, you write:

```sql
select * from {{ source('jaffle_shop', 'raw_customers') }}
```

**Why not just hardcode the table name?** Because `source()` gives dbt lineage tracking (it knows this model depends on that raw table), lets you test source freshness, and makes it a one-line change if the raw table ever moves.

## 5. Staging models — the first layer

Staging models are the entry point of transformation: one staging model per raw source table, doing light cleanup only — renaming columns, casting types, maybe basic filtering. No joins, no business logic yet.

Naming convention: `stg_<source>__<table>.sql` or simply `stg_customers.sql`.

Example concept (not final code):
```sql
select
    id as customer_id,
    first_name,
    last_name
from {{ source('jaffle_shop', 'raw_customers') }}
```

This is the foundation everything else in the project builds on.

## 6. `dbt run` vs `dbt build`

- `dbt run` — executes your models (turns SQL + Jinja into tables/views in Snowflake)
- `dbt build` — runs models **and** tests **and** snapshots **and** seeds, in the correct dependency order (you'll use this from Day 3 onward)

Today you'll mostly use `dbt run` since you don't have tests yet.

## 7. `dbt_project.yml` — the project's control file

This YAML file sits at the root of every dbt project and tells dbt:
- the project's name and version
- which dbt version it's compatible with
- default configs per folder (e.g. "everything in `models/staging/` materializes as a view")

You won't edit much of it today, but open it and read it — you'll come back to it on Day 2 when setting materializations folder-by-folder instead of model-by-model.

## 8. Folder-per-layer convention (why staging/intermediate/marts exist)

dbt doesn't force a folder structure, but the near-universal industry convention is:

```
staging      → 1:1 with a raw source table, light cleanup only
intermediate → joins/logic that's reused but not "final"
marts        → business-facing final tables, one concept per table (dim_customers, fct_orders)
```

The reason this convention exists: it keeps a clear line between "reflecting what the raw data says" (staging) and "answering a business question" (marts). If a source column gets renamed upstream, you fix it in *one* staging model instead of hunting through fifty downstream models. You'll only build staging today — intermediate and marts come Day 2.

## 9. Views vs tables — a first look (full depth on Day 2)

By default, staging models materialize as **views** (a saved query, recomputed each time it's queried — no storage cost, always fresh). Marts often materialize as **tables** (physically stored, faster to query, needs a rebuild to refresh). You don't need to configure anything today — dbt's default for staging is already correct — but notice this choice is configurable per model, which you'll explore properly on Day 2.

## 10. Snowflake concepts you're using without necessarily building

Two Snowflake ideas worth understanding today, since you set them up in Step 1:
- **Warehouse** — the compute engine that actually runs your SQL. It's separate from storage, and it can be paused/resumed (trial warehouses often auto-suspend — if `dbt run` hangs, check whether the warehouse is suspended).
- **Database → Schema → Table** hierarchy — Snowflake nests tables inside schemas inside databases. Your `profiles.yml` points dbt at a specific database + schema by default, which is where your models will land unless a model overrides it.

---

**End-of-day checkpoint:** you should be able to run `dbt run` and see 3 staging views/tables appear in your Snowflake schema, built on top of seeded raw data, all referenced through `source()`.