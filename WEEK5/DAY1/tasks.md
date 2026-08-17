# Day 1 — Task Checklist

**Time budget: 6 hours total**

Goal by end of day: dbt connected to Snowflake, raw data seeded, sources declared, 3 staging models running.

| Step | Task | Time |
|------|------|------|
| 1 | Snowflake setup | 30 min |
| 2 | Install dbt | 20 min |
| 3 | Initialize project | 20 min |
| 4 | Test connection | 15 min |
| 5 | Add seed data | 45 min |
| 6 | Declare sources | 30 min |
| 7 | Write 3 staging models | 90 min |
| 8 | Run and verify | 30 min |
| 9 | Review `dbt_project.yml` | 20 min |
| 10 | Add descriptions + source freshness | 30 min |
| — | Buffer / debugging | 30 min |

---

## Step 1: Snowflake setup (pre-work, verify first) — *30 min*

- [ ] Log in to your Snowflake trial account
- [ ] Create a database: `create database jaffle_shop_db;`
- [ ] Create a schema: `create schema jaffle_shop_db.raw;`
- [ ] Confirm you have a warehouse available (trial default is usually `COMPUTE_WH`)
- [ ] Note down: account identifier, username, password, role, warehouse, database

## Step 2: Install dbt — *20 min*

```bash
pip install dbt-snowflake
dbt --version
```
- [ ] Confirm `dbt-snowflake` adapter shows up in version output

## Step 3: Initialize the project — *20 min*

```bash
dbt init jaffle_shop_dbt
cd jaffle_shop_dbt
```
- [ ] When prompted, choose `snowflake` as the adapter
- [ ] Fill in account/user/password/role/warehouse/database/schema when asked
- [ ] This creates/updates `~/.dbt/profiles.yml` — open it and confirm your credentials are there

## Step 4: Test the connection — *15 min*

```bash
dbt debug
```
- [ ] All checks should say "Connection test: [OK connection ok]"
- [ ] If it fails, check account identifier format (often needs `.` replaced with `-` for Snowflake, e.g. `xy12345.ap-south-1` vs `xy12345-ap-south-1`)

## Step 5: Add seed data — *45 min*

- [ ] Download Jaffle Shop raw CSVs (`raw_customers.csv`, `raw_orders.csv`, `raw_payments.csv`) — from the [dbt Labs jaffle-shop-classic repo](https://github.com/dbt-labs/jaffle-shop-classic) `seeds/` folder
- [ ] Place them in your project's `seeds/` folder
- [ ] Create `raw_product_prices_v2.csv` yourself with columns: `product_id, product_name, price, valid_from` — 5-6 rows is enough
- [ ] Run:
```bash
dbt seed
```
- [ ] Confirm 4 tables now exist in Snowflake under your `raw` schema

## Step 6: Declare sources — *30 min*

- [ ] Create `models/staging/sources.yml`
- [ ] Declare a source named `jaffle_shop` pointing to your `raw` schema
- [ ] List tables: `raw_customers`, `raw_orders`, `raw_payments`

## Step 7: Write 3 staging models — *90 min*

In `models/staging/`, create:
- [ ] `stg_customers.sql` — select from `source('jaffle_shop', 'raw_customers')`, rename `id` → `customer_id`
- [ ] `stg_orders.sql` — select from `source('jaffle_shop', 'raw_orders')`, rename `id` → `order_id`
- [ ] `stg_payments.sql` — select from `source('jaffle_shop', 'raw_payments')`, rename `id` → `payment_id`

## Step 8: Run and verify — *30 min*

```bash
dbt run
```
- [ ] All 3 models run successfully (green output)
- [ ] Log into Snowflake and confirm 3 views (or tables) exist in your dev schema
- [ ] Spot-check: `select * from stg_customers limit 5;`

## Step 9: Review `dbt_project.yml` — *20 min*

- [ ] Open `dbt_project.yml` and read through it fully
- [ ] Find the `models:` config block and identify where staging's default materialization (`view`) is set
- [ ] Change the project `name` field to something identifiable (e.g. your own project name) and re-run `dbt run` to confirm nothing breaks
- [ ] Note (don't act yet): this is the file you'll edit Day 2 to set mart materializations

## Step 10: Add descriptions + a source freshness check — *30 min*

- [ ] In `sources.yml`, add a `description:` under each table (e.g. "Raw customer records from the app database")
- [ ] Add a `loaded_at_field` and basic `freshness:` block (warn after 24h, error after 48h) to at least one source table — this won't do much on seeded data, but write it so you understand the syntax
- [ ] Run:
```bash
dbt source freshness
```
- [ ] Read the output — note whether it passed/warned/failed and why

---

## End-of-day submission

Push your project to a GitHub repo (private is fine) with a commit message like `Day 1: setup, seeds, sources, staging models`. Share the repo link with the instructor.

**Stuck points to watch for:** Snowflake account identifier format, warehouse being suspended (resume it in Snowflake UI), and case-sensitivity of column names in CSVs vs SQL.