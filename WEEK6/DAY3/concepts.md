# Day 3 — Streams, Task DAGs, Snowpipe

## Where we're headed today

The first two days built a platform that works when we run SQL by hand. Today we make it react on
its own: a Stream watches `raw_orders` for changes, a Task DAG picks those changes up and processes
them into a summary table without us triggering it manually, and Snowpipe loads a new incoming file
the moment it lands — including handling the fact that this file's schema doesn't quite match what
we loaded on Day 1.

---

## 1. Streams — change tracking, not a copy of the data

A **Stream** doesn't store data itself. It sits on top of a table and records what changed
(inserted, updated, deleted rows) since it was last consumed. The moment we `SELECT` from a stream
or use it inside a `DML` statement that consumes it, that "since last consumed" pointer moves
forward — which means a stream can only be read meaningfully once per change, not queried
repeatedly like a table.

Two flavors matter here:

- **Standard stream** — tracks inserts, updates, and deletes, and tells us which is which via
  metadata columns (`METADATA$ACTION`, `METADATA$ISUPDATE`)
- **`APPEND_ONLY` stream** — tracks inserts only, ignores updates/deletes entirely, and is cheaper
  to maintain when we genuinely only care about new rows

We use a standard stream on `raw_orders`, because we want to know when a `status` changes (say,
`placed` → `shipped`), not just when a new order appears.

## 2. Task DAGs — one task depending on another, not a single scheduled job

A single `TASK` running one SQL statement on a schedule is the easy version. The real skill is a
**DAG**: a root task, and one or more child tasks that only run after the root succeeds, built with
`AFTER <root_task_name>` instead of their own `SCHEDULE`.

```
root_task (runs on schedule, consumes the stream)
     └── summary_task (runs only after root_task succeeds, builds the summary table)
```

This mirrors a real pipeline shape: "ingest/consume" as one concern, "aggregate/summarize" as a
separate downstream concern, each independently retryable. Tasks are created suspended by default
— we have to explicitly `ALTER TASK ... RESUME`, and the root of the DAG is the only one that needs
a `SCHEDULE`; children just need `AFTER`.

## 3. Snowpipe — the same COPY INTO, but triggered instead of run by hand

Snowpipe is not a different loading mechanism — under the hood it's still `COPY INTO` with the same
file format and stage concepts from Day 1. What's different is *what triggers it*: instead of us
running `COPY INTO` manually, a `PIPE` object runs it automatically when a new file shows up (via a
cloud storage event notification in production, or a manual `ALTER PIPE ... REFRESH` / stage upload
trigger for our purposes this week).

We check `PIPE_STATUS()` and the pipe's load history to confirm the file was picked up, the same
way we checked `COPY_HISTORY` on Day 1 — Snowpipe doesn't remove the need to verify a load, it just
removes the need to trigger it by hand.

## 4. Schema drift — the part that's actually hard today

`raw_orders_batch2.csv` doesn't match the file format we defined on Day 1. It has one new column
and one renamed column. This is deliberate, because clean re-loads of identical schemas teach
nothing about what actually breaks pipelines in practice.

We have two honest options, and we pick one and justify it in writing rather than just making the
load succeed by any means:

- **Widen the target table first** (`ALTER TABLE raw_orders ADD COLUMN ...`, handle the rename
  explicitly in the `COPY INTO` column list), so the schema is intentional and versioned
- **Load into a staging table with the new shape**, reconcile the rename there, then merge into
  `raw_orders` — safer when we don't fully trust the drift is a one-time thing

Either is defensible. What's not defensible is not noticing the drift until the load silently fails
or silently loads garbage into the wrong column — so step one today is *detecting* the drift before
touching anything, not reacting to a failure after the fact.

## 5. Stream staleness — the failure mode that doesn't announce itself

A stream's offset is only valid as long as the underlying table's change history covers it, which
is bounded by the table's `DATA_RETENTION_TIME_IN_DAYS`. If a stream goes unconsumed for longer
than that window, it becomes **stale** — Snowflake sets `STALE = TRUE` on it and it silently stops
being able to see further back than the retention boundary. A stale stream doesn't error loudly by
default; it just quietly returns less than it should, which is worse than an error because nothing
tells us to look.

Before consuming a stream in a task, it's worth checking `SYSTEM$STREAM_HAS_DATA('<stream_name>')`
— it tells us whether there's anything to process at all, which also means we can skip firing up
the warehouse for an empty run. Checking staleness (`SHOW STREAMS` and looking at the `stale`
column) is the other half: if a stream might go unconsumed for a while (a paused task, a long
outage), we either shorten the consuming schedule or lengthen the table's retention window
deliberately, not by accident.

## 6. Fan-out — one stream, two independent consumers

A stream can be read by more than one downstream process, as long as each reader consumes it in
its own transaction. We build a second, independent task off the *same* `orders_stream` — one that
tracks something different from the summary table (for example, flagging orders that moved straight
to `'delivered'` without ever showing `'shipped'`, a data-quality signal rather than a count). This
is the realistic shape of a data platform: one change stream, multiple concerns feeding off it,
each able to fail or fall behind independently without affecting the others.

## 7. Task failure handling — what happens when a scheduled task errors

Tasks fail: a bad row slips past `ON_ERROR = 'CONTINUE'` and breaks a downstream `MERGE`, a
warehouse is suspended and slow to resume, a transient network blip hits. By default a task keeps
retrying on its schedule forever, which can silently burn credits on a task that will never
succeed. `SUSPEND_TASK_AFTER_NUM_FAILURES` sets a ceiling — after N consecutive failures, Snowflake
suspends the task automatically instead of retrying indefinitely.

We also look at task-level history beyond just "did it run" — `TASK_HISTORY` includes an
`ERROR_MESSAGE` column, and a DAG's overall run can be inspected as a graph run (not just individual
task rows) via `INFORMATION_SCHEMA.TASK_HISTORY` filtered by `SCHEDULED_TIME`. We deliberately break
one task (point it at a table that doesn't exist, or similar), watch it fail on schedule, and
confirm we can read the actual error message rather than just seeing a red "FAILED" status with no
explanation.

## 8. Idempotency — why the summary task uses MERGE, not INSERT

If the root task's `MERGE` were an `INSERT` instead, running it twice against the same stream
content (say, after a manual re-trigger during testing) would double-count. `MERGE` with an
explicit match condition is idempotent in the sense that matters here: reprocessing the same
logical change updates the same row rather than creating a duplicate. This matters more than it
sounds — a scheduled task that occasionally gets triggered twice (manual testing, a retry after a
transient failure) is normal, and a pipeline that can't tolerate that is a fragile one.

---

## What "done" looks like at the end of today

- A standard Stream on `raw_orders`, demonstrated actually detecting an insert and a status update
  (not just created — proven, with the metadata columns showing which is which)
- A Task DAG: a root task consuming the stream on a schedule, and at least one child task (`AFTER`)
  building a summary table from what the root processed
- A second, independent task fanned out off the same stream, proving a stream can support more than
  one consumer
- `SYSTEM$STREAM_HAS_DATA` checked and explained, and stream staleness understood in terms of
  `DATA_RETENTION_TIME_IN_DAYS`
- A task deliberately made to fail, its `ERROR_MESSAGE` read from `TASK_HISTORY`, and
  `SUSPEND_TASK_AFTER_NUM_FAILURES` configured and explained
- The schema drift in `raw_orders_batch2.csv` detected deliberately before loading, one of the two
  handling approaches chosen and justified, and the file loaded successfully afterward
- Snowpipe configured against the batch-2 stage location, demonstrated triggering a load, and its
  pipe status / load history checked to confirm what happened

Tomorrow is performance and cost: clustering keys, caching, multi-cluster concurrency, and resource
monitors — tested against data large enough (TPC-H at SF10/SF100) that the differences are real,
not imagined.