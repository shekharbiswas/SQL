# Day 2 — Task Checklist

Run everything as `data_engineer` unless noted. Warehouse and schema context from Day 1 still apply.

```sql
USE ROLE data_engineer;
USE WAREHOUSE project_wh;
USE SCHEMA snowflake_platform_project.raw;
```

---

## 1. Load raw_order_items.json

- [ ] Build (or export) `raw_order_items.json` — one JSON document per order, each with an
      `items` array, and each item carrying its own `modifiers` array. Aim for a few thousand
      orders' worth of items, e.g.:

```json
{"order_id": 1, "items": [
  {"item_id": 101, "product": "Latte", "price": 4.50,
   "modifiers": [{"name": "oat milk", "price": 0.60}, {"name": "extra shot", "price": 0.80}]},
  {"item_id": 102, "product": "Croissant", "price": 3.20, "modifiers": []}
]}
```

```sql
CREATE FILE FORMAT json_ff TYPE = 'JSON' STRIP_OUTER_ARRAY = FALSE;

CREATE STAGE json_stage FILE_FORMAT = json_ff;
-- PUT the file onto json_stage from your terminal / Snowsight upload

CREATE OR REPLACE TABLE raw_order_items_variant (data VARIANT);

COPY INTO raw_order_items_variant
FROM @json_stage/raw_order_items.json
FILE_FORMAT = (FORMAT_NAME = json_ff);
```

**Checklist:**
- [ ] `raw_order_items_variant` loaded, row count matches number of JSON documents

---

## 2. Flatten both nesting levels

```sql
-- one level: items
SELECT
  data:order_id::INT                       AS order_id,
  item.value:item_id::INT                  AS item_id,
  item.value:product::STRING               AS product,
  item.value:price::FLOAT                  AS item_price
FROM raw_order_items_variant,
     LATERAL FLATTEN(input => data:items) AS item;

-- two levels: items, then modifiers within each item
SELECT
  data:order_id::INT                       AS order_id,
  item.value:item_id::INT                  AS item_id,
  item.value:product::STRING               AS product,
  modifier.value:name::STRING              AS modifier_name,
  modifier.value:price::FLOAT              AS modifier_price
FROM raw_order_items_variant,
     LATERAL FLATTEN(input => data:items) AS item,
     LATERAL FLATTEN(input => item.value:modifiers, OUTER => TRUE) AS modifier;
```

> `OUTER => TRUE` on the second flatten keeps items with an empty `modifiers` array (like the
> croissant above) instead of silently dropping them — check what happens if you leave it off.

**Checklist:**
- [ ] Single-level flatten (items) row count matches total item count across all orders
- [ ] Two-level flatten (modifiers) row count is correct, and items with zero modifiers are still
      present (confirm `OUTER => TRUE` behavior by comparing counts with and without it)

---

## 3. Views vs. materialized view

```sql
CREATE VIEW vw_order_item_totals AS
SELECT order_id, SUM(item_price) AS items_subtotal
FROM ( /* single-level flatten query from above */ ... )
GROUP BY order_id;

CREATE MATERIALIZED VIEW mv_order_item_totals AS
SELECT order_id, SUM(item_price) AS items_subtotal
FROM ( /* same query */ ... )
GROUP BY order_id;
```

**Checklist:**
- [ ] Both objects created and return matching results
- [ ] One or two written sentences: for *this* specific query, is the materialized view worth its
      storage + maintenance cost, or would the plain view have been the right call? Why?

---

## 4. Time Travel — recover from a bad UPDATE

```sql
-- note the current time first
SELECT CURRENT_TIMESTAMP();

-- deliberately break some data
UPDATE raw_orders SET order_total = 0 WHERE UNIFORM(0,1,RANDOM()) < 0.1;

-- confirm the damage
SELECT COUNT(*) FROM raw_orders WHERE order_total = 0;

-- recover using Time Travel, comparing against the pre-update state
SELECT COUNT(*) FROM raw_orders AT (OFFSET => -60*5) WHERE order_total = 0;

-- restore (pick one approach and justify it):
CREATE OR REPLACE TABLE raw_orders AS
SELECT * FROM raw_orders BEFORE (STATEMENT => '<query_id_of_the_bad_update>');
```

**Checklist:**
- [ ] Bad `UPDATE` run and confirmed (row count with `order_total = 0` is non-trivial)
- [ ] Table restored using `AT`/`BEFORE`, not `UNDROP`
- [ ] Row count of zeroed totals back to pre-damage level after restore

---

## 5. Zero-copy clone

```sql
CREATE TABLE dev_orders CLONE raw_orders;

-- confirm it's a full logical copy
SELECT COUNT(*) FROM dev_orders;

-- change the clone only
UPDATE dev_orders SET status = 'returned' WHERE order_id = 1;

-- confirm the original is untouched
SELECT status FROM raw_orders WHERE order_id = 1;
SELECT status FROM dev_orders WHERE order_id = 1;
```

**Checklist:**
- [ ] `dev_orders` row count matches `raw_orders` immediately after cloning
- [ ] Change to `dev_orders` confirmed not to appear in `raw_orders`

---

## 6. Check Time Travel retention window

```sql
SELECT DATA_RETENTION_TIME_IN_DAYS
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_NAME = 'RAW_ORDERS';
```

**Checklist:**
- [ ] Retention value recorded for `raw_orders`
- [ ] One sentence: if today's bad `UPDATE` had gone unnoticed for longer than this retention
      window, could we still have recovered it with Time Travel? What would we have needed instead?

---

## End-of-day proof

Before moving to Day 3, we should be able to show:
1. Two-level flatten output with correct row counts, including the `OUTER => TRUE` comparison
2. The materialized-view-vs-view decision written in our own words
3. The bad `UPDATE`, its detection, and the Time Travel recovery — with row counts before/after
4. The clone divergence test showing `raw_orders` and `dev_orders` disagree only where expected
5. `DATA_RETENTION_TIME_IN_DAYS` checked, with a note on the practical limit of Time Travel recovery

Tomorrow: Streams track what changed in `raw_orders`, a Task DAG processes those changes
automatically, and Snowpipe brings in the batch-2 file (with its schema drift) on autopilot.