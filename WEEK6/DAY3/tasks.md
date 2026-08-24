# Day 3 — Task Checklist

```sql
USE ROLE data_engineer;
USE WAREHOUSE project_wh;
USE SCHEMA snowflake_platform_project.raw;
```

---

## 1. Create a standard stream on raw_orders

```sql
CREATE STREAM orders_stream ON TABLE raw_orders;

-- generate some real change to detect
INSERT INTO raw_orders (order_id, customer_id, order_date, order_total, status)
VALUES (9999901, 123, CURRENT_DATE(), 42.50, 'placed');

UPDATE raw_orders SET status = 'shipped' WHERE order_id = 9999901;

-- inspect what the stream sees before consuming it
SELECT order_id, status, METADATA$ACTION, METADATA$ISUPDATE
FROM orders_stream
WHERE order_id = 9999901;
```

**Checklist:**
- [ ] Stream created on `raw_orders`
- [ ] Insert and update both visible in the stream, with `METADATA$ACTION`/`METADATA$ISUPDATE`
      correctly distinguishing them
- [ ] One sentence: what happens to these rows in the stream the next time we consume it in a task?

---

## 2. Build a Task DAG (root + child)

```sql
CREATE TABLE orders_status_summary (
  status STRING,
  order_count INT,
  last_updated TIMESTAMP
);

CREATE TASK root_process_orders_stream
  WAREHOUSE = project_wh
  SCHEDULE = '5 MINUTE'
AS
  MERGE INTO orders_status_summary tgt
  USING (
    SELECT status, COUNT(*) AS cnt
    FROM orders_stream
    GROUP BY status
  ) src
  ON tgt.status = src.status
  WHEN MATCHED THEN UPDATE SET order_count = src.cnt, last_updated = CURRENT_TIMESTAMP()
  WHEN NOT MATCHED THEN INSERT (status, order_count, last_updated)
    VALUES (src.status, src.cnt, CURRENT_TIMESTAMP());

CREATE TASK child_log_summary_change
  WAREHOUSE = project_wh
  AFTER root_process_orders_stream
AS
  INSERT INTO summary_change_log
  SELECT *, CURRENT_TIMESTAMP() FROM orders_status_summary;

-- tasks are created suspended
ALTER TASK child_log_summary_change RESUME;
ALTER TASK root_process_orders_stream RESUME;
```

> `summary_change_log` needs to exist first with matching columns — create it before the child
> task, matching `orders_status_summary` plus a timestamp column.

**Checklist:**
- [ ] Root task created with a `SCHEDULE`, consumes the stream
- [ ] Child task created with `AFTER <root_task>`, no schedule of its own
- [ ] Both resumed (root first is fine, order of `RESUME` doesn't matter, but both must be resumed
      or nothing runs)
- [ ] After waiting for at least one scheduled run (or manually via `EXECUTE TASK
      root_process_orders_stream`), confirm via `TASK_HISTORY` that the child ran *after* the root,
      not independently

```sql
SELECT name, state, scheduled_time, completed_time
FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY())
ORDER BY scheduled_time DESC
LIMIT 10;
```

---

## 3. Stream staleness and SYSTEM$STREAM_HAS_DATA

```sql
-- check whether there's anything to process before spinning up compute
SELECT SYSTEM$STREAM_HAS_DATA('orders_stream');

-- check current retention and stream staleness status
SHOW STREAMS LIKE 'orders_stream';
-- look at the STALE column in the result

SELECT DATA_RETENTION_TIME_IN_DAYS
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_NAME = 'RAW_ORDERS';
```

**Checklist:**
- [ ] `SYSTEM$STREAM_HAS_DATA` run before and after generating new changes, confirming it flips
      from `FALSE` to `TRUE`
- [ ] `STALE` column checked on the stream and `DATA_RETENTION_TIME_IN_DAYS` looked up for
      `raw_orders`
- [ ] One sentence: if this task were paused for a week, would the stream go stale first, or would
      the retention window survive it — and how do we know?

---

## 4. Fan-out — a second task off the same stream

```sql
CREATE TABLE order_status_anomalies (
  order_id INT,
  detail STRING,
  flagged_at TIMESTAMP
);

CREATE TASK root_flag_status_anomalies
  WAREHOUSE = project_wh
  SCHEDULE = '5 MINUTE'
AS
  INSERT INTO order_status_anomalies (order_id, detail, flagged_at)
  SELECT order_id, 'jumped to delivered without shipped', CURRENT_TIMESTAMP()
  FROM orders_stream
  WHERE status = 'delivered' AND METADATA$ACTION = 'INSERT';

ALTER TASK root_flag_status_anomalies RESUME;
```

**Checklist:**
- [ ] Second, independent task created reading from the *same* `orders_stream` as the summary task
- [ ] Both tasks confirmed running on their own schedules without interfering with each other
      (check `TASK_HISTORY` for both by name)
- [ ] One sentence: why does consuming the same stream from two independent tasks not cause either
      one to miss data the other already read?

---

## 5. Task failure handling

```sql
-- deliberately break a task by pointing it at something that doesn't exist
CREATE TASK broken_task
  WAREHOUSE = project_wh
  SCHEDULE = '5 MINUTE'
  SUSPEND_TASK_AFTER_NUM_FAILURES = 3
AS
  INSERT INTO this_table_does_not_exist SELECT * FROM orders_stream;

ALTER TASK broken_task RESUME;

-- after a few scheduled runs, check what happened
SELECT name, state, error_message, scheduled_time, completed_time
FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY())
WHERE name = 'BROKEN_TASK'
ORDER BY scheduled_time DESC;

SHOW TASKS LIKE 'broken_task';
-- confirm it auto-suspended after 3 failures
```

**Checklist:**
- [ ] `broken_task` created with `SUSPEND_TASK_AFTER_NUM_FAILURES = 3` and allowed to fail at least
      once (waiting for schedule, or triggered manually with `EXECUTE TASK`)
- [ ] Actual `ERROR_MESSAGE` text read from `TASK_HISTORY`, not just the `FAILED` status
- [ ] Confirmed the task auto-suspended after reaching the failure threshold
- [ ] `broken_task` dropped or fixed once the exercise is done, so it doesn't keep failing silently

---

## 6. Detect and handle schema drift in raw_orders_batch2.csv

- [ ] Before loading anything, open the file and compare its header row against `raw_orders`'
      columns. Write down exactly what's different (new column name, renamed column)
- [ ] Choose an approach:
  - **Option A — widen the target table:**
    ```sql
    ALTER TABLE raw_orders ADD COLUMN IF NOT EXISTS fulfillment_channel STRING;
    -- then in COPY INTO, map the renamed source column explicitly:
    COPY INTO raw_orders (order_id, customer_id, order_date, order_total, status, fulfillment_channel)
    FROM (SELECT $1, $2, $3, $4, $5, $6 FROM @raw_stage/raw_orders_batch2.csv)
    FILE_FORMAT = (FORMAT_NAME = csv_ff)
    ON_ERROR = 'CONTINUE';
    ```
  - **Option B — stage first, reconcile, then merge:**
    ```sql
    CREATE TABLE stg_orders_batch2 (order_id INT, customer_id INT, order_date DATE,
      order_total FLOAT, order_status STRING, channel STRING);
    COPY INTO stg_orders_batch2 FROM @raw_stage/raw_orders_batch2.csv
      FILE_FORMAT = (FORMAT_NAME = csv_ff) ON_ERROR = 'CONTINUE';
    -- reconcile the rename (order_status -> status) and merge into raw_orders explicitly
    ```
- [ ] One or two written sentences: which option was chosen and why, for this specific case

**Checklist:**
- [ ] Drift documented before loading
- [ ] Batch 2 loaded successfully using the chosen approach
- [ ] Row count of `raw_orders` (or the reconciled table) increased by the expected ~50K

---

## 7. Snowpipe on the batch-2 location

```sql
CREATE PIPE batch2_pipe
  AUTO_INGEST = FALSE  -- manual trigger for this exercise; AUTO_INGEST needs a cloud notification integration
AS
  COPY INTO raw_orders (order_id, customer_id, order_date, order_total, status, fulfillment_channel)
  FROM (SELECT $1, $2, $3, $4, $5, $6 FROM @raw_stage)
  FILE_FORMAT = (FORMAT_NAME = csv_ff)
  ON_ERROR = 'CONTINUE';

-- trigger it (simulating what a cloud notification would normally do)
ALTER PIPE batch2_pipe REFRESH;

-- check status and history
SELECT SYSTEM$PIPE_STATUS('batch2_pipe');

SELECT *
FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME => 'RAW_ORDERS',
  START_TIME => DATEADD(hour, -1, CURRENT_TIMESTAMP())
))
WHERE PIPE_NAME = 'BATCH2_PIPE';
```

**Checklist:**
- [ ] Pipe created referencing the correct file format and column mapping
- [ ] `REFRESH` triggered and confirmed via `SYSTEM$PIPE_STATUS`
- [ ] Load confirmed in `COPY_HISTORY` filtered by pipe name

---

## End-of-day proof

Before moving to Day 4, we should be able to show:
1. Stream output distinguishing an insert from an update via metadata columns
2. `TASK_HISTORY` showing the child task ran strictly after the root task
3. `SYSTEM$STREAM_HAS_DATA` flipping states, and the stream's `STALE` status checked against the
   table's retention window
4. Two independent tasks both consuming `orders_stream` without stepping on each other
5. A task deliberately broken, its real `ERROR_MESSAGE` read from `TASK_HISTORY`, and confirmation
   it auto-suspended after `SUSPEND_TASK_AFTER_NUM_FAILURES`
6. The written note on the schema drift and which handling approach was chosen
7. Pipe status and copy history confirming the batch-2 load happened through Snowpipe, not a manual
   `COPY INTO`

Tomorrow: clustering keys, caching, multi-cluster warehouse concurrency, and resource monitors —
tested against TPC-H at real scale.