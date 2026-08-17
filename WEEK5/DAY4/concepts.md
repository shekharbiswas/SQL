# Day 4 — Concepts (Learned by Doing)

Today's build: make one model incremental, write a custom macro, pull in a package, and run the entire project end-to-end with `dbt build`. This is the **last build day** — everything you learn today is aimed at having a complete, working project by end of day, so Day 5 is presentation prep only.

---

## 1. Incremental models — why they exist

So far, every model fully rebuilds from scratch on every `dbt run`. Fine for small data — but imagine `fct_orders` had 500 million rows. Rebuilding all of it every run wastes huge time and compute cost. **Incremental models** solve this: on the first run they build the full table, but on every run after, they only process *new or changed* rows and merge them in.

## 2. `is_incremental()` — the core building block

This is a Jinja function that's only `true` when: the model is configured as incremental, the target table already exists, and you're not doing a full-refresh. You wrap your "only new rows" filter in it:

```sql
{{
    config(
        materialized='incremental',
        unique_key='order_id'
    )
}}

select *
from {{ ref('int_orders_joined') }}

{% if is_incremental() %}
where order_date > (select max(order_date) from {{ this }})
{% endif %}
```

- On the **first run**, `is_incremental()` is false — the whole `select` runs, full table gets built
- On **later runs**, it's true — only rows newer than what's already in the table (`{{ this }}` refers to the model's own current table) get selected and merged in

## 3. `unique_key` and merge behavior

`unique_key` tells dbt how to identify "this is the same row, update it" vs "this is a new row, insert it." Without it, dbt just appends new rows, which can create duplicates if a row changes without a new primary key. With it, dbt merges — a real production concern this simulates well even on small data.

## 4. Full-refresh — the escape hatch

Incremental models can get "wrong" if logic changes are made after the fact (old rows won't have the new logic applied). The fix is a full rebuild:

```bash
dbt run --full-refresh --select fct_orders
```

This ignores incremental logic for one run and rebuilds the table completely — good to know for debugging, and worth demonstrating once so you understand why it matters.

## 5. Macros — reusable Jinja functions

A macro is a reusable chunk of Jinja/SQL, defined once, called anywhere — like a function. Defined in `macros/`:

```sql
{% macro cents_to_dollars(column_name) %}
    ({{ column_name }} / 100.0)
{% endmacro %}
```

Used in any model:
```sql
select {{ cents_to_dollars('amount_cents') }} as amount_dollars
from {{ ref('stg_payments') }}
```

Macros stop you from repeating the same SQL snippet across many models — change the logic once, it updates everywhere it's used.

## 6. Packages — reusing other people's macros

dbt has a package ecosystem (like npm/pip) for sharing macros across projects. The most common one, **`dbt_utils`**, ships pre-built macros for common needs — surrogate keys, date spines, pivoting, and generic tests beyond the built-in 4.

Installed via `packages.yml`:
```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: [">=1.0.0", "<2.0.0"]
```
Then:
```bash
dbt deps
```
Downloads it into `dbt_packages/`. A common use — generating a surrogate key:
```sql
select {{ dbt_utils.generate_surrogate_key(['customer_id', 'order_date']) }} as order_pk
```

## 7. `dbt build` — the full pipeline, one command

You've used `dbt run` and `dbt test` separately. `dbt build` runs seeds → snapshots → models → tests together in one dependency-respecting execution, and — importantly — **stops downstream models from running if an upstream test fails**. This is the command a real production job would run, and it's what you'll run today to prove the whole project works end to end.

```bash
dbt build
```

---

**End-of-day checkpoint (project complete):** one incremental model working correctly across two runs, one custom macro used in at least one model, `dbt_utils` installed and used at least once, and a full `dbt build` running clean from empty — this is your finished project. Day 5 is presentation prep only, no new building.