# Day 1 — Account Setup, Role Hierarchy, Warehouses, Loading at Scale

## Where we're headed today

By the end of today we have: a database and schema to work in, a warehouse sized to actually do
the work, a role hierarchy that mirrors how a real team would be set up (not just one admin doing
everything), and 1M+ rows of order data sitting in real tables — loaded the way data actually lands
in Snowflake, not hidden behind a tool.

Everything below is explained through what we're about to build, not as abstract theory.

---

## 1. Databases, schemas, warehouses — the three things everything else sits on

Snowflake splits **storage** and **compute** cleanly, and today is the first time this actually
matters to us:

- **Database → Schema → Table** is just a namespace hierarchy, like folders. We create one
  database for this project and a `RAW` schema to hold everything we load today.
- A **warehouse** is compute — the thing that actually runs our queries and loads. It costs
  nothing while suspended and starts billing the second a query touches it. This is why
  auto-suspend matters from minute one, not just on Day 4.

We're not sizing our warehouse generously today — an `X-SMALL` is enough to load 1M rows. We'll
feel the difference sizing makes later, once we're running queries against tens of millions of
TPC-H rows on Day 4. Today's lesson is just: compute and storage are separate, and we pay for
compute by the second it's running.

## 2. Role hierarchy — why one admin role is a red flag

The original plan for this kind of week is often "just do everything as `ACCOUNTADMIN`." We're not
doing that, because it teaches a habit that gets a real Snowflake account locked down or audited
badly. Instead we build a small hierarchy:

```
ACCOUNTADMIN
   └── SYSADMIN
         ├── DATA_ENGINEER      (can load, transform, create objects)
         └── ANALYST_READONLY   (can only SELECT)
```

Two things make this realistic rather than decorative:

- **Role inheritance** — a role granted to `SYSADMIN` is available to anyone with `SYSADMIN`, so
  we build our custom roles as children of `SYSADMIN`, not as siblings floating on their own.
- **Future grants** — normally a `GRANT SELECT ON TABLE x TO ROLE y` only applies to tables that
  already exist. `GRANT SELECT ON FUTURE TABLES IN SCHEMA raw TO ROLE analyst_readonly` means any
  table we create in `RAW` from this point on is automatically visible to that role — without us
  remembering to grant it every time. This is the difference between RBAC that works on day one
  and RBAC that quietly breaks by day thirty.

We will load all of today's data as `DATA_ENGINEER`, then briefly switch to `ANALYST_READONLY` to
confirm it really can read but not write. If it can write, our grants are wrong — that's the check.

## 3. Stages — the thing that actually gets skipped when people learn Snowflake through dbt

A **stage** is a named location Snowflake knows how to read files from. An **internal stage**
lives inside Snowflake itself — we `PUT` a local file onto it, then `COPY INTO` a table from it.

This is worth sitting with, because tools like `dbt seed` hide this entire step. In a real pipeline,
someone (or some process) has to get a file onto a stage before anything else can happen. Today we
do it by hand once, deliberately, so it's not a black box later.

## 4. File formats — telling Snowflake how to read what we hand it

A **file format object** tells `COPY INTO` how to parse a file: delimiter, header row, how to treat
quotes, how to treat NULLs. We define one CSV file format today and reuse it for every CSV we load
this week (including the schema-drifted batch on Day 3).

## 5. COPY INTO — and what happens when a row is bad

`COPY INTO <table> FROM <stage> FILE_FORMAT = ...` is the real load statement. Two options we use
today that most quick tutorials skip:

- **`VALIDATION_MODE`** — a dry run. It tells us what *would* load and what *would* fail, without
  writing anything. We run this first on every file today.
- **`ON_ERROR`** — what to do when a row is malformed. `'SKIP_FILE'` aborts the whole file on the
  first bad row (safe but blunt). `'CONTINUE'` skips just the bad row and loads everything else
  (useful, but means we must go check what got skipped).

We're loading a large enough dataset today that we deliberately include a handful of malformed
rows in it, on purpose — so we get real practice reading `COPY_HISTORY` and load error output
instead of only ever seeing a clean, silent success.

## 6. Generating our own data volume

Instead of being handed a small static file, we generate `raw_orders` ourselves using:

- **`GENERATOR(ROWCOUNT => n)`** — produces `n` rows out of nothing, the engine for synthesizing
  volume
- **`SEQ8()`** — a sequence, used as our row/order id
- **`UNIFORM(min, max, RANDOM())`** — random values in a range, used to spread order dates and
  amounts realistically instead of everything landing on the same day

This matters beyond today: it's a pattern for generating realistic test data on any future project,
without needing someone to hand us a file at all.

## 7. Checking grants directly — SHOW GRANTS, not just trusting the GRANT statement

We wrote the `GRANT` statements ourselves on purpose, but writing a grant and confirming it took
effect are two different things — a typo'd role name or a grant issued against the wrong object
fails silently in the sense that nothing complains, the permission just isn't there. `SHOW GRANTS`
lets us check both directions:

```sql
SHOW GRANTS TO ROLE data_engineer;      -- everything this role can do
SHOW GRANTS ON TABLE raw_orders;        -- everyone who can touch this specific table
```

We use this today to independently verify our Day 1 grants rather than only relying on the
role-switch test — two different ways of confirming the same thing catches mistakes that one way
alone would miss.

---

## What "done" looks like at the end of today

- Database, `RAW` schema, and a right-sized warehouse exist
- `SYSADMIN` → `DATA_ENGINEER` / `ANALYST_READONLY` role hierarchy is in place, with future grants
  confirmed working (not just granted — actually tested)
- `raw_customers` (~50K rows) and `raw_orders` (1M+ rows) exist as real tables, loaded via internal
  stage + `COPY INTO`, generated using `GENERATOR`/`SEQ8`/`UNIFORM`
- At least one malformed row was deliberately hit, handled with `ON_ERROR`, and explained in our
  own words — not just "it worked"
- Grants independently verified with `SHOW GRANTS`, not just the role-switch test

Tomorrow we bring in `raw_order_items.json` — nested arrays, `VARIANT`, `FLATTEN`, plus views, Time
Travel, and cloning.