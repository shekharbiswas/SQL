# Day 2 — Task Checklist

**Time budget: 6 hours total**

Goal by end of day: intermediate model + 2 mart models built, materializations configured at folder and model level, `ref()` used throughout, Jinja practiced.

| Step | Task | Time |
|------|------|------|
| 1 | Review yesterday's DAG | 15 min |
| 2 | Build `int_orders_joined.sql` | 60 min |
| 3 | Set folder-level materializations in `dbt_project.yml` | 30 min |
| 4 | Build `dim_customers.sql` | 60 min |
| 5 | Build `fct_orders.sql` | 60 min |
| 6 | Practice Jinja (`if`, `set`, `var`) | 45 min |
| 7 | Override a materialization explicitly | 20 min |
| 8 | Run, verify, inspect DAG | 40 min |
| — | Buffer / debugging | 50 min |

---

## Step 1: Review yesterday's DAG — *15 min*

```bash
dbt docs generate
dbt docs serve
```
- [ ] Open the docs site and look at the **Lineage Graph** for your 3 staging models
- [ ] Confirm you understand: staging models have no dbt-model dependencies (only source dependencies)

## Step 2: Build `int_orders_joined.sql` — *60 min*

- [ ] Create `models/intermediate/int_orders_joined.sql`
- [ ] Join `ref('stg_orders')` with `ref('stg_payments')` on `order_id`
- [ ] Aggregate payments per order (sum of amount) so you get one row per order with a `total_amount` column
- [ ] Do **not** add a materialization config yet — leave it to the folder default (Step 3)

## Step 3: Set folder-level materializations — *30 min*

- [ ] Open `dbt_project.yml`
- [ ] Under the `models:` block, set:
  - `staging` → `view`
  - `intermediate` → `ephemeral`
  - `marts` → `table`
- [ ] Run `dbt run` and confirm `int_orders_joined` does **not** appear as its own object in Snowflake (ephemeral models compile as CTEs — check the compiled SQL in `target/compiled/`)

## Step 4: Build `dim_customers.sql` — *60 min*

- [ ] Create `models/marts/dim_customers.sql`
- [ ] Select from `ref('stg_customers')`
- [ ] Add a calculated column, e.g. `full_name` (concat first + last name) — this is your first bit of real transformation logic
- [ ] Confirm it uses `ref()`, not raw table names

## Step 5: Build `fct_orders.sql` — *60 min*

- [ ] Create `models/marts/fct_orders.sql`
- [ ] Select from `ref('int_orders_joined')`
- [ ] Join in `customer_id` via `ref('stg_orders')` if not already present
- [ ] Include: `order_id`, `customer_id`, `order_date`, `total_amount`
- [ ] This model should now depend on both an intermediate model and a staging model — check the DAG shows this correctly

## Step 6: Practice Jinja — *45 min*

- [ ] In any one model, add a `{% set %}` variable and use it in a `where` clause (e.g. a minimum order amount threshold)
- [ ] Add a `{% if %}` block controlled by a `var()` — define the var's default in `dbt_project.yml` under a `vars:` block
- [ ] Run with the var overridden from the CLI: `dbt run --vars '{"your_var": true}'` and confirm behavior changes

## Step 7: Override a materialization explicitly — *20 min*

- [ ] Pick one mart (e.g. `fct_orders`) and add an explicit `{{ config(materialized='table') }}` at the top, even though it matches the folder default — this is just to practice the syntax you'll need for incremental models on Day 4

## Step 8: Run, verify, inspect DAG — *40 min*

```bash
dbt run
dbt docs generate
dbt docs serve
```
- [ ] All models run green
- [ ] In Snowflake, confirm `dim_customers` and `fct_orders` exist as **tables**, and `int_orders_joined` does **not** exist as a separate object
- [ ] In the docs lineage graph, confirm the chain: `raw sources → staging → intermediate → marts` is visually correct

---

## End-of-day submission

Push to GitHub with commit message `Day 2: intermediate + mart models, materializations, Jinja`. Share the repo link with the instructor.

**Stuck points to watch for:** ephemeral models won't show up in Snowflake at all — that's expected, not a bug. If a mart model errors with "column not found," check you're selecting the right column names post-join. `--vars` must be valid JSON on the command line — watch for quote escaping issues on Windows terminals.