# Day 3 — Concepts (Learned by Doing)

Today's build: add tests to catch bad data, generate documentation, and build your first snapshot to track slowly changing data. Every concept below is taught *through* these steps.

---

## 1. Why testing matters in dbt

Up to now you've trusted that your models produce correct data. Tests turn that trust into something checkable — a test is just a SQL query that should return **zero rows** if everything is fine. dbt runs these automatically and fails loudly if data breaks an assumption (e.g. a duplicate customer_id, a null order_id).

## 2. Generic tests — the four built-in ones

Defined in YAML, not SQL, applied directly to columns:

| Test | Checks |
|---|---|
| `unique` | No duplicate values in this column |
| `not_null` | No nulls in this column |
| `accepted_values` | Column only contains values from a given list |
| `relationships` | Every value in this column exists in another table's column (referential integrity) |

Example, applied in a `.yml` file next to your models:

```yaml
models:
  - name: dim_customers
    columns:
      - name: customer_id
        tests:
          - unique
          - not_null
      - name: status
        tests:
          - accepted_values:
              values: ['active', 'inactive']
```

## 3. Custom (singular) tests

Sometimes a business rule doesn't fit the 4 generic tests — e.g. "total_amount should never be negative." A singular test is just a `.sql` file in the `tests/` folder that returns rows **only when something is wrong**:

```sql
select *
from {{ ref('fct_orders') }}
where total_amount < 0
```

If this query returns any rows, the test fails. No config needed — dbt automatically picks up every `.sql` file in `tests/` as a test.

## 4. Running tests

```bash
dbt test
```

Runs every test in the project. You can also scope it: `dbt test --select dim_customers` runs only tests attached to that model. Tests run in dependency order too — no different from models.

## 5. Documentation — descriptions + the docs site

dbt lets you describe every model and column directly in YAML:

```yaml
models:
  - name: dim_customers
    description: "One row per customer, with calculated lifetime attributes."
    columns:
      - name: customer_id
        description: "Primary key, unique per customer."
```

Running `dbt docs generate` builds a full static site — searchable model list, column descriptions, and (most importantly) an **interactive DAG/lineage graph** showing exactly how data flows from raw sources through staging, intermediate, and marts. `dbt docs serve` opens it locally.

This isn't just for looks — in a real team, this docs site is how a new analyst figures out what a table means and where it came from without asking anyone.

## 6. Snapshots — tracking change over time (SCD Type 2)

Normal dbt models show you *current* state — if a source value changes, the old value is gone. **Snapshots** solve this: they detect row changes between runs and keep a full history, adding `dbt_valid_from` / `dbt_valid_to` columns automatically. This is the classic **Slowly Changing Dimension Type 2** pattern.

A snapshot is defined in `snapshots/` folder:

```sql
{% snapshot product_prices_snapshot %}

{{
    config(
      target_schema='snapshots',
      unique_key='product_id',
      strategy='timestamp',
      updated_at='valid_from',
    )
}}

select * from {{ source('jaffle_shop', 'raw_product_prices_v2') }}

{% endsnapshot %}
```

- `unique_key` — identifies a row across runs
- `strategy` — `timestamp` (compare an `updated_at` column) or `check` (compare specific columns for any change)
- Each `dbt snapshot` run compares current source data to the last snapshot and inserts new history rows only when something changed

This is exactly why yesterday's synthetic `raw_product_prices_v2.csv` exists — you'll update a price in that CSV, re-seed, and re-snapshot to *see* history being captured.

## 7. `dbt build` — running everything together

From today onward, prefer `dbt build` over `dbt run`. It runs seeds, snapshots, models, and tests together, in correct dependency order, and stops a downstream model from running if an upstream test fails. This is much closer to how a real pipeline behaves.

---

**End-of-day checkpoint:** `dbt test` passes with at least 5 generic tests + 1 custom test in place, `dbt docs generate` produces a working lineage graph with descriptions, and `dbt snapshot` has captured at least 2 versions of a product price after you changed it.