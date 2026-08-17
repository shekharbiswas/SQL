# Day 4 — Task Checklist

**Time budget: 6 hours total**

**This is the final build day.** By the end of today, the project must be fully complete — Day 5 (ending with a 2 PM presentation) is prep and polish only, no new modeling work.

Goal by end of day: one incremental model, one custom macro, `dbt_utils` package installed and used, full `dbt build` running clean, project README written.

| Step | Task | Time |
|------|------|------|
| 1 | Convert `fct_orders` to incremental | 60 min |
| 2 | Test incremental behavior across two runs | 40 min |
| 3 | Try `--full-refresh` | 20 min |
| 4 | Write a custom macro | 40 min |
| 5 | Install and use `dbt_utils` | 40 min |
| 6 | Run full `dbt build` from a clean state | 40 min |
| 7 | Final project README | 45 min |
| 8 | Final cleanup + repo polish | 40 min |
| — | Buffer / debugging | 35 min |

---

## Step 1: Convert `fct_orders` to incremental — *60 min*

- [ ] Add `config(materialized='incremental', unique_key='order_id')` to `fct_orders.sql`
- [ ] Add an `is_incremental()` block filtering on `order_date > (select max(order_date) from {{ this }})`
- [ ] Run `dbt run --select fct_orders` — this is the first run, full table build
- [ ] Confirm the table exists in Snowflake with all rows

## Step 2: Test incremental behavior across two runs — *40 min*

- [ ] Add 1-2 new rows to `raw_orders.csv` with a later `order_date` than anything currently in the table
- [ ] Re-seed: `dbt seed`
- [ ] Run again: `dbt run --select fct_orders`
- [ ] Query the table in Snowflake — confirm row count went up by exactly the new rows (not a full duplicate rebuild)
- [ ] Check the dbt run logs — incremental runs should say something like "merge" rather than "create or replace table"

## Step 3: Try `--full-refresh` — *20 min*

```bash
dbt run --full-refresh --select fct_orders
```
- [ ] Confirm the table rebuilds completely
- [ ] Understand (write a one-line note for yourself) when you'd need to do this in a real project — e.g. after changing incremental model logic

## Step 4: Write a custom macro — *40 min*

- [ ] Create `macros/custom_macros.sql`
- [ ] Write one macro — a good simple one is a reusable "cents to dollars" or "trim and lowercase a string" macro
- [ ] Use it in at least one model
- [ ] Run `dbt run` and confirm it compiles and works

## Step 5: Install and use `dbt_utils` — *40 min*

- [ ] Create `packages.yml` at the project root with the `dbt_utils` package pinned
- [ ] Run `dbt deps`
- [ ] Use `dbt_utils.generate_surrogate_key()` in one model (e.g. to create a surrogate key on `fct_orders` from `order_id` + `customer_id`)
- [ ] Optionally use one `dbt_utils` generic test (e.g. `dbt_utils.not_constant` or `dbt_utils.expression_is_true`) on any column
- [ ] Run `dbt build` and confirm the new column/test works

## Step 6: Run full `dbt build` from a clean state — *40 min*

- [ ] Drop your dev schema in Snowflake (or use a fresh schema) to simulate a clean environment
- [ ] Update `profiles.yml`/target schema if needed
- [ ] Run:
```bash
dbt build
```
- [ ] Confirm seeds, snapshots, models, and tests all run successfully in one command, in correct dependency order
- [ ] This full green run is your proof the project works end-to-end — screenshot it for the presentation

## Step 7: Final project README — *45 min*

- [ ] Write (or finalize) your project's own `README.md` inside the dbt project repo, covering:
  - What the project does (one paragraph)
  - Dataset used (Jaffle Shop + synthetic price data)
  - Tools used (dbt Core, Snowflake)
  - How to run it (`dbt deps`, `dbt seed`, `dbt build`)
  - A summary list of dbt concepts demonstrated (sources, seeds, staging/intermediate/marts, materializations, tests, docs, snapshots, incremental models, macros, packages)
  - Your DAG screenshot embedded

## Step 8: Final cleanup + repo polish — *40 min*

- [ ] Remove any leftover debug/test code or commented-out blocks
- [ ] Confirm folder structure matches the plan from the weekly README
- [ ] Confirm `.gitignore` excludes `target/`, `dbt_packages/`, `logs/`, and `profiles.yml` if it's inside the repo
- [ ] Final commit and push

---

## End-of-day submission

Push to GitHub with commit message `Day 4: incremental model, macros, packages, full dbt build — project complete`. Share the final repo link with the instructor.

**Project is now complete.** Nothing new gets built on Day 5 — Day 5 morning is for rehearsing your walkthrough and polishing the presentation itself, ahead of the 2 PM presentation slot.

**Stuck points to watch for:** incremental models can silently "miss" rows if the `is_incremental()` filter condition is too strict or too loose — always sanity-check row counts after a second run. `dbt deps` requires internet access to pull from the dbt package hub — run it early in case of network issues.