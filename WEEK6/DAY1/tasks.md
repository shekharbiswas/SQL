# Day 1 — Task Checklist

Work through these in order. Run everything as `ACCOUNTADMIN` only where marked — everything else
should be run as the role we've just created, to prove the grants actually work.

---

## 1. Database, schema, warehouse

```sql
-- as ACCOUNTADMIN
CREATE DATABASE IF NOT EXISTS snowflake_platform_project;
CREATE SCHEMA IF NOT EXISTS snowflake_platform_project.raw;

CREATE WAREHOUSE IF NOT EXISTS project_wh
  WAREHOUSE_SIZE = 'XSMALL'
  AUTO_SUSPEND = 60
  AUTO_RESUME = TRUE
  INITIALLY_SUSPENDED = TRUE;
```

**Checklist:**
- [ ] Database and `raw` schema exist
- [ ] Warehouse exists with `AUTO_SUSPEND = 60` and `AUTO_RESUME = TRUE`
- [ ] Can explain in one sentence why `INITIALLY_SUSPENDED = TRUE` matters for cost

---

## 2. Role hierarchy and future grants

```sql
-- as ACCOUNTADMIN
CREATE ROLE IF NOT EXISTS data_engineer;
CREATE ROLE IF NOT EXISTS analyst_readonly;

GRANT ROLE data_engineer TO ROLE sysadmin;
GRANT ROLE analyst_readonly TO ROLE sysadmin;
GRANT ROLE data_engineer TO USER <your_username>;
GRANT ROLE analyst_readonly TO USER <your_username>;

GRANT USAGE ON DATABASE snowflake_platform_project TO ROLE data_engineer;
GRANT USAGE ON DATABASE snowflake_platform_project TO ROLE analyst_readonly;
GRANT USAGE ON SCHEMA snowflake_platform_project.raw TO ROLE data_engineer;
GRANT USAGE ON SCHEMA snowflake_platform_project.raw TO ROLE analyst_readonly;
GRANT USAGE ON WAREHOUSE project_wh TO ROLE data_engineer;
GRANT USAGE ON WAREHOUSE project_wh TO ROLE analyst_readonly;

GRANT CREATE TABLE, CREATE STAGE, CREATE FILE FORMAT
  ON SCHEMA snowflake_platform_project.raw TO ROLE data_engineer;

-- future grants: anything created in raw from now on is auto-readable by analyst_readonly
GRANT SELECT ON FUTURE TABLES IN SCHEMA snowflake_platform_project.raw
  TO ROLE analyst_readonly;
```

**Checklist:**
- [ ] `data_engineer` and `analyst_readonly` both exist under `sysadmin`
- [ ] Future grant statement run *before* any table is created
- [ ] Switched role to `analyst_readonly` and confirmed `SELECT` works once tables exist below,
      and `INSERT`/`CREATE` fail (screenshot the failure — this is the actual proof, not the grant
      statement itself)

---

## 3. File format and internal stage

```sql
-- as DATA_ENGINEER
USE ROLE data_engineer;
USE WAREHOUSE project_wh;
USE SCHEMA snowflake_platform_project.raw;

CREATE FILE FORMAT csv_ff
  TYPE = 'CSV'
  FIELD_DELIMITER = ','
  SKIP_HEADER = 1
  NULL_IF = ('', 'NULL')
  EMPTY_FIELD_AS_NULL = TRUE;

CREATE STAGE raw_stage
  FILE_FORMAT = csv_ff;
```

**Checklist:**
- [ ] File format created with header skipping and NULL handling
- [ ] Stage created referencing that file format

---

## 4. Generate raw_customers (~50K rows) and raw_orders (1M+ rows)

```sql
CREATE OR REPLACE TABLE raw_customers AS
SELECT
  SEQ8() + 1                                       AS customer_id,
  'customer_' || (SEQ8() + 1)                      AS customer_name,
  ARRAY_CONSTRUCT('DE','FR','ES','IT','NL')[UNIFORM(0,4,RANDOM())] AS country
FROM TABLE(GENERATOR(ROWCOUNT => 50000));

CREATE OR REPLACE TABLE raw_orders AS
SELECT
  SEQ8() + 1                                            AS order_id,
  UNIFORM(1, 50000, RANDOM())                           AS customer_id,
  DATEADD(day, -UNIFORM(0, 730, RANDOM()), CURRENT_DATE()) AS order_date,
  ROUND(UNIFORM(5, 50000, RANDOM()) / 100.0, 2)         AS order_total,
  ARRAY_CONSTRUCT('placed','shipped','delivered','returned')[UNIFORM(0,3,RANDOM())] AS status
FROM TABLE(GENERATOR(ROWCOUNT => 1000000));
```

**Checklist:**
- [ ] `raw_customers` has ~50K rows
- [ ] `raw_orders` has 1,000,000+ rows
- [ ] `order_date` spans roughly 2 years, not all identical (spot check with `SELECT
      MIN(order_date), MAX(order_date) FROM raw_orders`)

> Note: these two tables are generated directly, not loaded via `COPY INTO` — that's intentional,
> this step is about volume generation. Section 5 below is where we practice the real stage +
> `COPY INTO` load path, using a file we actually place on the stage.

---

## 5. Load a CSV through the stage properly (with a deliberately bad row)

- [ ] Export a small CSV of ~500 additional "late-arriving" orders (any tool — even a quick
      spreadsheet), and **deliberately corrupt 3–5 rows** (a text value where a number is expected,
      a missing required field, an extra column on one row)
- [ ] `PUT` the file onto `raw_stage`
- [ ] Run `COPY INTO raw_orders FROM @raw_stage/<file> FILE_FORMAT = (FORMAT_NAME = csv_ff)
      VALIDATION_MODE = 'RETURN_ERRORS';` first — read the output, don't just glance at it
- [ ] Run the real load with `ON_ERROR = 'CONTINUE'`
- [ ] Query `COPY_HISTORY` (or the load result) to confirm exactly how many rows loaded vs. skipped,
      and write one sentence explaining what happened to the bad rows

```sql
SELECT *
FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME => 'RAW_ORDERS',
  START_TIME => DATEADD(hour, -1, CURRENT_TIMESTAMP())
));
```

---

## 6. Verify grants independently with SHOW GRANTS

```sql
SHOW GRANTS TO ROLE data_engineer;
SHOW GRANTS TO ROLE analyst_readonly;
SHOW GRANTS ON TABLE raw_orders;
```

**Checklist:**
- [ ] Output of `SHOW GRANTS TO ROLE analyst_readonly` matches what we expect from the future
      grant (SELECT only, on the tables that exist so far)
- [ ] `SHOW GRANTS ON TABLE raw_orders` confirms both roles appear with the expected privilege
      level, cross-checked against the role-switch test from section 2

---

## End-of-day proof

Before moving to Day 2, we should be able to show:
1. A screenshot of `analyst_readonly` successfully running `SELECT` and failing on `INSERT`
2. Row counts for `raw_customers` and `raw_orders`
3. The `VALIDATION_MODE` output showing the bad rows *before* they were skipped
4. One sentence, in our own words, on the difference between `ON_ERROR = 'SKIP_FILE'` and
   `'CONTINUE'` — and which we'd choose for a nightly production load, and why
5. `SHOW GRANTS` output confirming both roles' actual privileges match what was intended

Tomorrow: `raw_order_items.json`, `VARIANT`/`FLATTEN` across nested arrays, views, Time Travel, and
zero-copy cloning.