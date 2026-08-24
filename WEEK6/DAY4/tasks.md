# Day 4 — Task Checklist

```sql
USE ROLE data_engineer;
USE WAREHOUSE project_wh;
```

Today mostly reads `SNOWFLAKE_SAMPLE_DATA` — no loading required, it's already available in any
Snowflake account.

---

## 1. Baseline — measure before touching anything

```sql
SELECT SYSTEM$CLUSTERING_INFORMATION(
  'snowflake_sample_data.tpch_sf10.lineitem', '(l_shipdate)'
);
```

**Checklist:**
- [ ] `average_depth` value recorded from the baseline (unclustered) state
- [ ] One sentence: based on this number alone, would you expect a `WHERE l_shipdate = ...` query
      to prune well or poorly right now?

---

## 2. Apply and verify a clustering key

```sql
CREATE OR REPLACE TABLE lineitem_clustered
  CLUSTER BY (l_shipdate)
AS
SELECT * FROM snowflake_sample_data.tpch_sf10.lineitem;

-- reclustering can take time; check clustering quality after it settles
SELECT SYSTEM$CLUSTERING_INFORMATION('lineitem_clustered', '(l_shipdate)');

-- compare a filtered query's partition scan stats before vs after
SELECT COUNT(*) FROM snowflake_sample_data.tpch_sf10.lineitem
WHERE l_shipdate BETWEEN '1995-01-01' AND '1995-01-31';

SELECT COUNT(*) FROM lineitem_clustered
WHERE l_shipdate BETWEEN '1995-01-01' AND '1995-01-31';
```

**Checklist:**
- [ ] `average_depth` on `lineitem_clustered` recorded and compared against the baseline
- [ ] Partition pruning compared between the two tables for the same filtered query (check "Partitions
      scanned" vs "Partitions total" in the query profile for each)
- [ ] One or two written sentences: was clustering actually worth it here, given the reclustering
      cost — would you make the same call on a table this size in production?

---

## 3. Result cache vs. warehouse cache vs. cold read

```sql
-- Run 1: cold baseline (note the timing)
SELECT l_returnflag, COUNT(*), SUM(l_extendedprice)
FROM snowflake_sample_data.tpch_sf10.lineitem
GROUP BY l_returnflag;

-- Run 2: identical query again immediately (result cache)
SELECT l_returnflag, COUNT(*), SUM(l_extendedprice)
FROM snowflake_sample_data.tpch_sf10.lineitem
GROUP BY l_returnflag;

-- Run 3: a DIFFERENT query against the SAME table, warehouse still warm (warehouse cache)
SELECT l_linestatus, COUNT(*), AVG(l_discount)
FROM snowflake_sample_data.tpch_sf10.lineitem
GROUP BY l_linestatus;

-- Force a cold state, then repeat Run 1
ALTER WAREHOUSE project_wh SUSPEND;
ALTER WAREHOUSE project_wh RESUME;

SELECT l_returnflag, COUNT(*), SUM(l_extendedprice)
FROM snowflake_sample_data.tpch_sf10.lineitem
GROUP BY l_returnflag;
```

**Checklist:**
- [ ] All four timings recorded (cold, result-cache hit, warehouse-cache-assisted different query,
      cold again after forced suspend/resume)
- [ ] Result-cache run confirmed near-instant regardless of table size
- [ ] Warehouse-cache run correctly identified as faster than cold but not a result-cache hit (it's
      a different query, so it can't be)
- [ ] One sentence: why did suspending and resuming the warehouse erase the speed advantage even
      though the query text in Run 4 was identical to Run 1?

---

## 4. Scale up — one query, bigger single warehouse

```sql
CREATE WAREHOUSE IF NOT EXISTS scale_up_test_wh WAREHOUSE_SIZE = 'XSMALL' AUTO_SUSPEND = 60;

USE WAREHOUSE scale_up_test_wh;
-- run a genuinely heavy single query (e.g. multi-table TPC-H join at SF10) and time it
ALTER WAREHOUSE scale_up_test_wh SUSPEND;

ALTER WAREHOUSE scale_up_test_wh SET WAREHOUSE_SIZE = 'LARGE';
ALTER WAREHOUSE scale_up_test_wh RESUME;
-- run the SAME heavy query again and time it
```

**Checklist:**
- [ ] Same heavy query timed on `XSMALL` and again on `LARGE`, both cold (suspend/resume between)
- [ ] Timing improvement recorded
- [ ] One sentence: does this test tell us anything about handling more *concurrent* users? Why not?

---

## 5. Scale out — multi-cluster, concurrent queries

```sql
CREATE WAREHOUSE IF NOT EXISTS concurrency_test_wh
  WAREHOUSE_SIZE = 'XSMALL'
  WAREHOUSE_TYPE = 'STANDARD'
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 1
  AUTO_SUSPEND = 60;

-- fire several different TPC-H queries at this warehouse back-to-back from multiple sessions/tabs,
-- and note total wall-clock time until all finish

ALTER WAREHOUSE concurrency_test_wh SET MAX_CLUSTER_COUNT = 4;

-- repeat the exact same burst of concurrent queries
```

**Checklist:**
- [ ] Same concurrent query burst run against `MAX_CLUSTER_COUNT = 1` and again with `= 4`
- [ ] Query queuing observed on the single-cluster run (check `WAREHOUSE_LOAD_HISTORY` or the
      Snowsight warehouse activity view for queued queries)
- [ ] Total completion time compared between the two runs
- [ ] One sentence: which of today's two experiments (scale up vs. scale out) would you reach for
      to fix "our dashboard is slow for one analyst" vs. "our dashboard times out when ten people
      open it at 9am"?

---

## 6. Read a query profile for spillage

```sql
-- pick one of the heavier queries already run above, find it in QUERY_HISTORY
SELECT query_id, query_text, total_elapsed_time, bytes_spilled_to_local_storage,
       bytes_spilled_to_remote_storage
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY())
ORDER BY total_elapsed_time DESC
LIMIT 5;
```

**Checklist:**
- [ ] Query profile opened (Snowsight) for the slowest query run today
- [ ] `bytes_spilled_to_local_storage` / `bytes_spilled_to_remote_storage` checked — spillage present
      or ruled out
- [ ] One or two written sentences: if spillage was present, would a bigger warehouse (more memory)
      or a different query shape be the right fix — and if there was no spillage, what does that
      tell us about where the time actually went instead?

---

## 7. Resource monitor with multiple thresholds

```sql
CREATE RESOURCE MONITOR project_week_monitor
  WITH CREDIT_QUOTA = 20
  FREQUENCY = DAILY
  START_TIMESTAMP = IMMEDIATELY
  TRIGGERS
    ON 50 PERCENT DO NOTIFY
    ON 75 PERCENT DO NOTIFY
    ON 100 PERCENT DO SUSPEND
    ON 110 PERCENT DO SUSPEND_IMMEDIATE;

ALTER WAREHOUSE project_wh SET RESOURCE_MONITOR = project_week_monitor;
ALTER WAREHOUSE scale_up_test_wh SET RESOURCE_MONITOR = project_week_monitor;
ALTER WAREHOUSE concurrency_test_wh SET RESOURCE_MONITOR = project_week_monitor;
```

**Checklist:**
- [ ] Resource monitor created with at least two `NOTIFY` thresholds and both a `SUSPEND` and
      `SUSPEND_IMMEDIATE` threshold
- [ ] Attached to all three warehouses used today
- [ ] One or two written sentences: why `SUSPEND` at 100% and `SUSPEND_IMMEDIATE` at 110% rather
      than just `SUSPEND_IMMEDIATE` at 100% — what's the actual trade-off being made here?

---

## 8. Check the real cost of today

```sql
SELECT warehouse_name, SUM(credits_used) AS total_credits
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
WHERE start_time >= DATEADD(hour, -6, CURRENT_TIMESTAMP())
GROUP BY warehouse_name
ORDER BY total_credits DESC;
```

**Checklist:**
- [ ] Credit usage pulled per warehouse for today's session
- [ ] One sentence: which single experiment today (clustering, scale-up, scale-out, or caching
      tests) cost the most, and does that match what you expected before running it?
- [ ] `scale_up_test_wh` and `concurrency_test_wh` dropped or resized back down, since they were
      created purely for today's comparisons

---

## End-of-day proof

Before moving to Day 5, we should be able to show:
1. `average_depth` before and after clustering, plus the pruning comparison on a filtered query
2. Four timings distinguishing cold read, result cache, warehouse cache, and cold-again
3. Scale-up vs. scale-out timings, each tied to the scenario it actually solves
4. A query profile screenshot with spillage identified or explicitly ruled out
5. The resource monitor with its threshold ladder, attached to all warehouses used today
6. Actual credit consumption pulled from `WAREHOUSE_METERING_HISTORY`, with the most expensive
   experiment named

Tomorrow: pull all five days together into a polished README, an architecture diagram, and a
5–10 minute presentation — including an honest account of what broke along the way.