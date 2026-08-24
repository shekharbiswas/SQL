# Day 4 — Performance & Cost at Scale

## Where we're headed today

The first three days built a platform that works correctly. Today is about whether it works
*efficiently* — and that question is meaningless on a 1M-row table. Everything today runs against
TPC-H at SF10 and SF100 (tens to hundreds of millions of rows) specifically because clustering,
caching, and warehouse sizing only show real, honest differences at that scale. We finish with
resource monitors, because none of this matters if it also means an unbounded bill.

---

## 1. Micro-partitions — the thing every performance concept today sits on

Snowflake automatically splits every table into micro-partitions (roughly 50–500MB of compressed
data each) as it's loaded, and stores metadata about the min/max values in each partition for every
column. A query with a filter on a well-organized column can skip (prune) entire micro-partitions
without reading them at all. Everything else today — clustering, caching, warehouse sizing — is
really just different angles on "how do we make pruning work in our favor, and what do we do when
it can't."

## 2. Clustering keys — reordering data so pruning actually works

Data lands roughly in load order. If we constantly filter on a column that has nothing to do with
load order (say, `l_shipdate` on a huge TPC-H `lineitem` table that was bulk-loaded all at once),
every micro-partition might contain a wide spread of dates, so a date filter can't prune much of
anything — Snowflake has to scan most of the table regardless of how selective the filter looks.

A **clustering key** tells Snowflake to keep reorganizing the table so that the values of that
column stay co-located within micro-partitions. This isn't free — reclustering costs credits — so
it's only worth it for large, frequently-filtered tables where the query pattern is stable. We use
`SYSTEM$CLUSTERING_INFORMATION` to *measure* clustering quality before deciding to add a key, not
just add one because the syntax is easy:

```sql
SELECT SYSTEM$CLUSTERING_INFORMATION('snowflake_sample_data.tpch_sf10.lineitem', '(l_shipdate)');
```

The `average_depth` figure in that output is the number worth watching — lower is better organized.
We apply a clustering key, wait for reclustering, and re-check the same function to see the number
actually move.

## 3. Query result cache vs. warehouse cache — two different kinds of "fast the second time"

These get conflated constantly, so today we deliberately separate them:

- **Query result cache** — if the exact same SQL text runs again within 24 hours and the underlying
  data hasn't changed, Snowflake returns the cached result *without touching a warehouse at all*.
  This is why the second run of an identical query can return in under a second even against a
  100M-row table — and why demoing "warehouse speed" by re-running the same query twice is
  accidentally lying about what's fast.
- **Warehouse (local disk / SSD) cache** — data recently read by a warehouse stays in that
  warehouse's local storage. A *different* query against the *same* data can benefit from this even
  though it's not a result-cache hit, because the raw data is already sitting on local disk instead
  of being re-fetched from remote storage. This cache disappears the moment the warehouse suspends.

We test both honestly: same query twice (result cache), then a *different* query against the same
table on a still-running warehouse (warehouse cache), then the same query again right after the
warehouse was forced to suspend and resume (neither cache helps, cold read).

## 4. Warehouse sizing — scale up vs. scale out, and why they solve different problems

**Scaling up** (`XSMALL` → `LARGE`, etc.) gives one query more compute to finish faster — more
threads, more memory, more local disk. This helps a single expensive query.

**Scaling out** (multi-cluster warehouses) adds more *copies* of the same warehouse size, each
handling separate queries in parallel. This helps *concurrency* — many people/queries hitting the
warehouse at once — and does nothing for how long any single query takes.

Mixing these up is the single most common warehouse-sizing mistake: making a warehouse bigger to
handle more concurrent users (wrong lever — that's what multi-cluster is for), or adding
multi-cluster to make one slow query faster (also wrong — that's what a bigger single warehouse is
for). We test both directly today so the difference isn't theoretical.

## 5. Multi-cluster warehouses — concurrency under real simulated load

A multi-cluster warehouse defined with `MIN_CLUSTER_COUNT` / `MAX_CLUSTER_COUNT` automatically
spins up additional clusters when queries start queuing, and suspends them again once load drops.
We simulate concurrent load by firing several TPC-H queries at the same warehouse in quick
succession from multiple sessions, then compare total wall-clock completion time on a
single-cluster warehouse against a multi-cluster one under the same load.

## 6. Reading a query profile — where time actually goes

The Query Profile (accessible per-query in Snowsight, or via `QUERY_HISTORY` /
`GET_QUERY_OPERATOR_STATS`) breaks a query down into its execution steps and shows, per step, how
much time was spent and — critically — whether any step **spilled to local or remote disk** because
it needed more memory than the warehouse had available. Spilling is often the real cause of a slow
query, and it's invisible unless we actually open the profile rather than just looking at total
runtime. We deliberately pick one of our bigger TPC-H queries, open its profile, and find (or rule
out) spillage before touching warehouse size at all — sizing up to fix a problem that isn't actually
about size wastes money without fixing anything.

## 7. Resource monitors — thresholds, not just a single limit

A resource monitor isn't one number — it supports multiple thresholds against a credit quota, each
with its own action: `NOTIFY` at 50%, another `NOTIFY` at 75%, and `SUSPEND` (let running queries
finish, block new ones) or `SUSPEND_IMMEDIATE` (kill running queries too) at 100%. Choosing
`SUSPEND` vs `SUSPEND_IMMEDIATE` is a real trade-off — `SUSPEND_IMMEDIATE` guarantees the spend cap
but can kill a query mid-write, while `SUSPEND` is gentler but can slightly overshoot the quota
while in-flight queries finish. We build a monitor with both a notify and a suspend threshold and
explain, in writing, which suspend behavior we chose and why.

## 8. Checking the actual bill — WAREHOUSE_METERING_HISTORY

Everything above is inferred from behavior; `ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY` (and
`WAREHOUSE_LOAD_HISTORY` for queueing) shows the actual credits consumed per warehouse over time.
We check this after today's testing to see, concretely, what today's experiments cost — turning
"caching saves money" from a claim into a number we measured ourselves.

---

## What "done" looks like at the end of today

- A clustering key applied to a large TPC-H table, with `SYSTEM$CLUSTERING_INFORMATION` measured
  before and after, showing `average_depth` actually improve
- Result cache and warehouse cache tested separately and correctly distinguished, with a cold-read
  baseline for comparison — three different timings, not two
- Scale-up (bigger single warehouse, one query) and scale-out (multi-cluster, concurrent queries)
  each tested against the scenario they actually solve, with timings recorded for both
- A query profile opened for at least one large query, spillage identified or ruled out, and a
  written note on what was actually slow before any warehouse resizing was attempted
- A resource monitor with a notify threshold and a suspend threshold, with `SUSPEND` vs
  `SUSPEND_IMMEDIATE` chosen deliberately and explained
- `WAREHOUSE_METERING_HISTORY` checked to see the real credit cost of today's testing

Tomorrow: polish the README, put together an architecture diagram, and prepare a 5–10 minute
presentation of the whole platform — including what broke along the way.