# dbt Project Week — Data Engineering with dbt + Snowflake

## Overview

This is a 5-day, project-based dbt bootcamp. Students learn **every major dbt concept by building one real project** — no standalone theory lectures. By Friday, each student has a working dbt project connected to Snowflake, built on the **Jaffle Shop** dataset, complete with staging models, marts, tests, documentation, snapshots, and incremental models — and presents it.

**Structure:** 4 build days + 1 presentation/polish day.

| Day | Focus | Mode |
|-----|-------|------|
| Day 1 | Setup + Sources + Seeds + Staging models | Build |
| Day 2 | Marts + Jinja + Materializations | Build |
| Day 3 | Testing + Documentation + Snapshots (SCD) | Build |
| Day 4 | Incremental models + Macros/Packages + Full `dbt build` | Build |
| Day 5 | Polish, README, DAG walkthrough, presentation | Present |

---

## Why Snowflake (instead of DuckDB)

Since students will present a project, using a real cloud warehouse makes the deliverable closer to an industry setup and gives real experience with:
- Snowflake trial account creation, roles, warehouses, databases, schemas
- `profiles.yml` connection configuration for a cloud warehouse
- Query cost/warehouse-size awareness (a genuine job-relevant skill)

**Setup needed before Day 1 (assign as pre-work):**
1. Sign up for a [Snowflake free trial](https://signup.snowflake.com/) (30-day / $400 credit)
2. Note down: account identifier, username, password, role (`ACCOUNTADMIN` is fine for a trial), warehouse name, database name
3. Install Python 3.9+ and `dbt-snowflake` (`pip install dbt-snowflake`)
4. Install a code editor (VS Code recommended) + Git

---

## Dataset: Jaffle Shop

We use dbt Labs' official **Jaffle Shop** sample dataset (raw `customers`, `orders`, `payments`, plus `products`, `stores`, `supplies` in the newer version). It's small, realistic, and was purpose-built by dbt Labs to teach dbt concepts — staging → intermediate → marts flows naturally out of it.

To simulate **slowly changing dimensions** for the snapshot lesson (Day 3), we inject one synthetic CSV (`raw_product_prices_v2.csv`) with updated prices — this gives students real change-data to snapshot without needing a second dataset.

Source data is loaded into Snowflake as raw tables (via `dbt seed` for CSVs, or pre-loaded raw tables — instructor's choice, both approaches covered).

---

## Concepts covered (mapped across the week)

- Project setup & Snowflake connection (`dbt init`, `profiles.yml`)
- Sources (`sources.yml`, `source()`, freshness)
- Seeds (`dbt seed`)
- Staging / intermediate / mart layering
- Materializations: view, table, incremental, ephemeral
- Jinja templating & `ref()` vs `source()`
- Generic + custom (singular) tests
- Documentation (`dbt docs generate` + descriptions)
- Snapshots (SCD Type 2)
- Incremental models (`is_incremental()`, unique_key, merge strategy)
- Macros & packages (`dbt_utils`)
- Full pipeline run (`dbt build`)

---

## Repo structure students will end up with

```
jaffle_shop_dbt/
├── README.md
├── dbt_project.yml
├── packages.yml
├── profiles.yml           (not committed — local only)
├── seeds/
│   └── raw_product_prices_v2.csv
├── models/
│   ├── staging/
│   │   ├── stg_customers.sql
│   │   ├── stg_orders.sql
│   │   └── stg_payments.sql
│   ├── intermediate/
│   │   └── int_orders_joined.sql
│   ├── marts/
│   │   ├── dim_customers.sql
│   │   ├── fct_orders.sql
│   │   └── mart_customer_lifetime_value.sql
│   └── sources.yml
├── snapshots/
│   └── product_prices_snapshot.sql
├── macros/
│   └── custom_macros.sql
└── tests/
    └── singular_test_example.sql
```

---

## Daily deliverables (what "done" looks like each day)

- **Day 1:** dbt connected to Snowflake, seeds loaded, sources declared, 3 staging models running and passing `dbt run`
- **Day 2:** at least 2 mart models built on top of staging, correct materializations chosen and justified
- **Day 3:** tests passing (`dbt test` green), docs site generated locally, one snapshot model capturing price history
- **Day 4:** one incremental model working correctly on a second run, one custom macro used, one package installed and used, full `dbt build` runs clean
- **Day 5:** polished README, DAG screenshot/walkthrough, 5–10 min presentation covering what was built and why

---

## Day-wise files

Each day has two companion files:
- `dayN_concepts.md` — the concepts for that day, explained through the project (no dry theory)
- `dayN_task.md` — the hands-on task/checklist to complete that day

Next: Day 1 concepts + task file.