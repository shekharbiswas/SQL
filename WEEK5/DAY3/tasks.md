# Day 3 — Task Checklist

**Time budget: 6 hours total**

Goal by end of day: generic + custom tests passing, docs site generated with descriptions, a working snapshot capturing price history.

| Step | Task | Time |
|------|------|------|
| 1 | Add generic tests to staging models | 45 min |
| 2 | Add generic tests to mart models | 45 min |
| 3 | Write a custom singular test | 40 min |
| 4 | Run and fix tests | 40 min |
| 5 | Add descriptions to models + columns | 45 min |
| 6 | Generate and explore docs site | 30 min |
| 7 | Build the price snapshot | 60 min |
| 8 | Trigger and verify a change in snapshot history | 45 min |
| — | Buffer / debugging | 30 min |

---

## Step 1: Add generic tests to staging models — *45 min*

- [ ] Create/open `models/staging/schema.yml` (or extend `sources.yml`)
- [ ] Add `unique` + `not_null` tests on `customer_id` in `stg_customers`
- [ ] Add `unique` + `not_null` tests on `order_id` in `stg_orders`
- [ ] Add `not_null` on `payment_id` in `stg_payments`

## Step 2: Add generic tests to mart models — *45 min*

- [ ] Create `models/marts/schema.yml`
- [ ] Add `unique` + `not_null` on `customer_id` in `dim_customers`
- [ ] Add `unique` + `not_null` on `order_id` in `fct_orders`
- [ ] Add a `relationships` test on `fct_orders.customer_id` → `dim_customers.customer_id`
- [ ] If you have a status/type column anywhere, add an `accepted_values` test on it

## Step 3: Write a custom singular test — *40 min*

- [ ] Create `tests/assert_total_amount_not_negative.sql`
- [ ] Write a query selecting from `fct_orders` where `total_amount < 0`
- [ ] (Optional second test) Create one more singular test of your choice — e.g. checking order dates aren't in the future

## Step 4: Run and fix tests — *40 min*

```bash
dbt test
```
- [ ] All tests pass (green)
- [ ] If any fail, inspect the failing rows (dbt shows a compiled SQL you can run directly in Snowflake) and fix the upstream model logic, not the test

## Step 5: Add descriptions to models + columns — *45 min*

- [ ] In every `schema.yml`, add a `description:` for each model (staging, intermediate if not ephemeral, marts)
- [ ] Add column-level `description:` for at least the primary key and 2 other columns per model

## Step 6: Generate and explore docs site — *30 min*

```bash
dbt docs generate
dbt docs serve
```
- [ ] Open the docs site
- [ ] Click into `fct_orders` and confirm your descriptions show up
- [ ] Explore the full lineage graph — trace a path from a raw source all the way to a mart
- [ ] Take a screenshot of the DAG — you'll need this for Friday's presentation

## Step 7: Build the price snapshot — *60 min*

- [ ] Create `snapshots/product_prices_snapshot.sql`
- [ ] Use the `raw_product_prices_v2` seed as the source, `timestamp` strategy, `unique_key='product_id'`
- [ ] Run:
```bash
dbt snapshot
```
- [ ] Confirm a new table appears in your `snapshots` schema in Snowflake with `dbt_valid_from` / `dbt_valid_to` columns
- [ ] Query it and confirm every row currently has `dbt_valid_to = null` (meaning "currently valid")

## Step 8: Trigger and verify a change in snapshot history — *45 min*

- [ ] Edit `raw_product_prices_v2.csv` — change the price of one product and update its `valid_from` date
- [ ] Re-seed: `dbt seed`
- [ ] Re-snapshot: `dbt snapshot`
- [ ] Query the snapshot table again — confirm the old row now has a `dbt_valid_to` timestamp, and a new row exists with `dbt_valid_to = null`
- [ ] This is SCD Type 2 in action — be ready to explain this table in your presentation

---

## End-of-day submission

Push to GitHub with commit message `Day 3: tests, docs, snapshot`. Share the repo link with the instructor, along with your DAG screenshot.

**Stuck points to watch for:** `relationships` test needs the referenced model's column to actually be unique, or it'll behave oddly — double check `dim_customers.customer_id` passed its `unique` test first. Snapshot `strategy='timestamp'` requires the `updated_at` column to actually change — if you edit the price but forget to update `valid_from` in the CSV, no new history row will appear.