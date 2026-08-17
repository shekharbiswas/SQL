# Day 2 — Concepts 

Today's build: turn yesterday's staging models into real business-facing tables — intermediate models, dimension/fact marts, Jinja templating, and a proper look at materializations. Every concept below is taught *through* these steps.

---

## 1. Intermediate models — the "in-between" layer

Intermediate models sit between staging and marts. They hold logic that's reused or complex enough to deserve its own model, but isn't yet a final business-facing table — for example, joining `stg_orders` with `stg_payments` to get one row per order with its total payment amount.

Naming convention: `int_<what_it_does>.sql`, e.g. `int_orders_joined.sql`.

**Why not just write one giant SQL query per mart?** Because breaking logic into intermediate steps makes each model easier to test, debug, and reuse across multiple marts. If both `fct_orders` and `mart_customer_lifetime_value` need "orders joined with payments," you write that join once in `int_orders_joined` instead of twice.

## 2. `ref()` — how models refer to each other

Just like `source()` points at raw tables, `ref()` points at other dbt models:

```sql
select * from {{ ref('stg_orders') }}
```

This is the single most important function in dbt. It does two things:
- Resolves to the correct table/view name and schema automatically (so you never hardcode names)
- Builds the **DAG** (dependency graph) — dbt uses every `ref()` call to figure out what depends on what, and therefore what order to run models in

If model B uses `ref('A')`, dbt guarantees A runs before B. You never have to manage run order yourself.

## 3. Marts — dimension and fact tables

Marts are the final, business-facing layer — the tables analysts and BI tools actually query. Two common shapes, borrowed from dimensional modeling:

- **Dimension tables** (`dim_customers`) — descriptive attributes about an entity (name, signup date, location)
- **Fact tables** (`fct_orders`) — measurable events, usually with foreign keys to dimensions and numeric metrics (order amount, order date, customer_id)

Today you'll build `dim_customers` and `fct_orders` on top of your staging and intermediate models.

## 4. Materializations, properly

A materialization is *how* dbt turns a `select` statement into something in your warehouse. Four types:

| Materialization | What it does | When to use |
|---|---|---|
| **view** | Saved query, recomputed on every read | Staging models — cheap, always fresh |
| **table** | Physically stored, rebuilt fully each run | Marts — faster reads, worth the storage/compute cost |
| **incremental** | Table that only processes *new/changed* rows on each run | Large fact tables where full rebuilds are slow (full depth on Day 4) |
| **ephemeral** | Not built in the warehouse at all — inlined as a CTE into whatever references it | Lightweight intermediate models you never query directly |

You set this with a config block at the top of a model:

```sql
{{ config(materialized='table') }}

select ...
```

Or set a default for an entire folder in `dbt_project.yml` — you saw this yesterday; today you'll actually change it, e.g. setting everything in `models/marts/` to materialize as `table` by default.

## 5. Jinja basics — the templating behind dbt

dbt models are SQL + **Jinja** (a Python templating language). You've already used two Jinja functions (`ref()`, `source()`) — today you go a level deeper:

- `{{ }}` — outputs a value (a function call, a variable)
- `{% %}` — control flow (if/for loops, no output)
- `{% set my_var = 'value' %}` — define a variable inside a model

Example — a simple `if` to conditionally filter:
```sql
select *
from {{ ref('stg_orders') }}
{% if var('exclude_test_orders', false) %}
where is_test_order = false
{% endif %}
```

This is what makes dbt models more powerful than plain SQL views — you can template repeated logic instead of copy-pasting it.

## 6. `dbt_project.yml` materialization config (folder-level defaults)

Yesterday you just read this file. Today you'll edit the `models:` block to set folder-level defaults, e.g.:

```yaml
models:
  jaffle_shop_dbt:
    staging:
      +materialized: view
    intermediate:
      +materialized: ephemeral
    marts:
      +materialized: table
```

This means you don't need a `config()` block in every single model — only when a model needs to *override* the folder default.

---

**End-of-day checkpoint:** you should have `int_orders_joined` (ephemeral), `dim_customers` (table), and `fct_orders` (table) all running via `dbt run`, connected through `ref()` calls, with materializations set both by folder default and by explicit override in at least one model.