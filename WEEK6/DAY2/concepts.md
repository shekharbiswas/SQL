# Day 2 — Semi-Structured Data, Views, Time Travel, Cloning

## Where we're headed today

Yesterday we loaded clean, tabular data. Today the data isn't tabular at all — it's nested JSON,
the kind that actually shows up from an app's event log or an API response. We also start treating
our tables like they can be safely broken and fixed, which is the whole point of Time Travel and
cloning: today's mistakes become tomorrow's non-issues.

---

## 1. VARIANT — Snowflake's "I don't know the shape yet" type

A `VARIANT` column holds semi-structured data (JSON, in our case) without forcing us to define a
rigid schema up front. We load `raw_order_items.json` into a table with a single `VARIANT` column,
then query into it using two bits of path notation:

- `:` reaches into an object by key — `data:order_id`
- `[]` reaches into an array by index — `data:items[0]`

Our JSON isn't flat — each order has an array of line items, and each line item has its own array
of modifiers (size, add-ons). That's an array nested inside an array, which is exactly the shape
that trips people up if they've only ever seen a single `FLATTEN()` call.

## 2. FLATTEN — turning nested arrays into rows, twice

`FLATTEN()` takes an array and produces one row per element. For a single level of nesting, one
`FLATTEN()` is enough. For our data — items nested inside orders, modifiers nested inside items —
we need to flatten twice, joining each `FLATTEN()` result back with `LATERAL`:

```
orders → FLATTEN(items)      → one row per line item
       → FLATTEN(modifiers)  → one row per modifier, still attached to its order and item
```

The `LATERAL` join is what lets the second `FLATTEN()` see the output of the first. This is the
actual skill today — not flattening one level (that's easy), but keeping the two levels correctly
joined back to their parent order without losing or duplicating rows. We check our work by
comparing row counts before and after each flatten step.

## 3. Views vs. Materialized Views — and why one of them can be a cost mistake

A **view** is just a saved query — it re-runs against live data every time it's queried, no storage
cost beyond the definition itself.

A **materialized view** actually stores the result and keeps it updated automatically as the base
table changes. That sounds strictly better, but it isn't:

- It costs storage
- Snowflake automatically re-runs background maintenance to keep it in sync, which costs compute
  *even if nobody ever queries it*
- It only helps when the underlying query is expensive **and** queried often **and** the base data
  doesn't change so often that maintenance cost outweighs the savings

We build one of each today, but the exercise is really about writing down, in our own words, why
a materialized view is or isn't the right call for that specific query — not just proving we know
the syntax.

## 4. Time Travel — recovering from a mistake, not just a deleted object

Every tutorial shows `UNDROP TABLE`. That's the easy case. The harder, more realistic case is: we
ran an `UPDATE` that was wrong — not deleted, just wrong — and the table still exists with bad data
sitting in it right now. Time Travel handles this too, using `AT`/`BEFORE`:

```sql
SELECT * FROM raw_orders AT (OFFSET => -60*5);  -- state from 5 minutes ago
```

We deliberately run a bad `UPDATE` on `raw_orders` today (something like zeroing out a chunk of
`order_total`), confirm the damage, then recover using a Time Travel query rather than an `UNDROP`.
This is the version of the skill we'd actually reach for in production, because most real mistakes
are wrong writes, not dropped tables.

## 5. Zero-copy cloning — a full dev copy that costs almost nothing

`CREATE TABLE dev_orders CLONE raw_orders` creates a full logical copy of a 1M+ row table instantly,
sharing the same underlying micro-partitions. Storage cost is effectively zero *until* one of the
two tables changes — only then does Snowflake start storing the diverged data separately.

We clone `raw_orders` into a "dev" copy, make a change to the dev copy only, and confirm the
original is untouched. This is the pattern that makes safe experimentation possible on a real
production-sized table without needing to reload a million rows just to test something.

## 6. Time Travel retention — how far back "back" actually goes

Time Travel isn't infinite. How far back `AT`/`BEFORE` can reach is bounded by the table's
`DATA_RETENTION_TIME_IN_DAYS` — 1 day by default on standard accounts, configurable higher per
table or per account edition. Recovering from yesterday's mistake works fine; recovering from a
mistake made three weeks ago doesn't, unless retention was deliberately set higher. We check this
setting today rather than assuming Time Travel will always be there when we need it:

```sql
SELECT DATA_RETENTION_TIME_IN_DAYS
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_NAME = 'RAW_ORDERS';
```

This also connects back to the stream staleness concept we'll hit later in the week — a stream's
usable history and a table's Time Travel window are governed by the same underlying setting.

---

## What "done" looks like at the end of today

- `raw_order_items.json` loaded into a `VARIANT` column and flattened across both nesting levels
  (items, then modifiers), with row counts sanity-checked at each step
- One standard view and one materialized view built, with a written sentence on why each was the
  right (or wrong) choice for that query
- A deliberate bad `UPDATE` run against `raw_orders`, then recovered using Time Travel `AT`/`BEFORE`
  — not `UNDROP`
- A zero-copy clone of `raw_orders` created, modified independently, and confirmed not to affect
  the original
- `DATA_RETENTION_TIME_IN_DAYS` checked on `raw_orders`, with a one-sentence note on how far back
  we could actually recover if today's mistake had happened a week ago instead

Tomorrow: Streams, a Task DAG, and Snowpipe — turning today's static tables into a pipeline that
reacts to new and changing data on its own.