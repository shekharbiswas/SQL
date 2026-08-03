# Day 3 Concepts: Recursive CTEs, Query Performance Basics & Pivoting

**Estimated reading time: 55 min**

Days 1 and 2 were one continuous idea: comparing each row to nearby rows, without a self-join or correlated subquery. Today covers three separate topics that don't share that thread, but each one explains something used earlier in the course without fully covering how it works — and each one shows up constantly in production SQL.

**Recursive CTEs** handle data that references itself — something Days 1 and 2 quietly depended on without ever showing you how it was built. **Reading `EXPLAIN` plans** answers a question you've probably already wondered about this course: why does one version of a query run faster than another that returns the same result? And **pivoting/unpivoting** solves a reshaping problem — rows into columns, and back — that's genuinely common in reporting and doesn't fit neatly into anything covered so far.

Tomorrow, Day 4 introduces dbt itself. Today is the last day of writing standalone `.sql` files before that — worth noticing, since two of today's three topics (a reusable recursive date-spine query, and the mechanics of *why* a query is slow) are exactly the kind of thing a dbt project organizes on purpose.

---

## Recursive CTEs: Data That References Itself

Every CTE so far has been non-recursive: define a named subquery once, reference it once (or a few times) later in the statement. A **recursive CTE** references *itself* inside its own definition, building up a result set iteration by iteration until nothing new is produced.

The syntax has two parts, joined by `UNION ALL`:

```sql
WITH RECURSIVE cte_name AS (
  -- Anchor member: runs once, defines the starting point
  SELECT ...

  UNION ALL

  -- Recursive member: references cte_name itself, runs repeatedly
  SELECT ...
  FROM cte_name
  JOIN ...
)
SELECT * FROM cte_name;
```

The **anchor member** runs once and produces the starting rows. The **recursive member** then runs repeatedly, each time operating on the rows produced by the *previous* iteration — not the whole growing result set, just the newest batch — and feeding its output back in. This continues until an iteration produces zero new rows, at which point the recursion stops on its own.

## A Motivating Example You've Already Been Using: The Date Spine

Every day this week, you've queried `daily_series` — one row per calendar date, with no gaps, going back years. That table didn't appear from nowhere. Generating a complete, gap-free sequence of dates is a classic recursive CTE use case, and it's worth seeing how it's actually built:

```sql
WITH RECURSIVE date_spine AS (
  SELECT DATE '2019-01-01' AS calendar_date  -- anchor: the first date

  UNION ALL

  SELECT calendar_date + INTERVAL 1 DAY
  FROM date_spine
  WHERE calendar_date < DATE '2025-12-31'    -- recursive member: keeps adding a day
)
SELECT calendar_date FROM date_spine;
```

The anchor produces exactly one row: January 1st, 2019. The recursive member then takes that one row, adds a day, checks the `WHERE` condition, and produces January 2nd — which becomes the input to the *next* iteration, producing January 3rd, and so on, until the date would exceed the cutoff and the recursion naturally stops. This is precisely the mechanism that makes Week 2, Day 2's gap-detection work possible in the first place: you can only find a missing date by comparing your real data against a spine that's guaranteed to have no gaps of its own, and that spine has to come from somewhere.

## A Second Example: Following a Bike Across Multiple Trips

Recursive CTEs are also the right tool whenever one row's output is the next row's input — a genuinely hierarchical or chained relationship, not just "rows near each other" the way Days 1–2 handled it. A single bike making several trips in a day, where each trip's start station is the previous trip's end station, is exactly that shape:

```sql
WITH RECURSIVE bike_chain AS (
  SELECT
    bike_id,
    trip_id,
    start_station,
    end_station,
    start_time,
    1 AS chain_position
  FROM trips t
  WHERE bike_id = 'B00214'
    AND start_time = (
      SELECT MIN(start_time) FROM trips WHERE bike_id = 'B00214' AND DATE(start_time) = '2024-06-03'
    )

  UNION ALL

  SELECT
    t.bike_id,
    t.trip_id,
    t.start_station,
    t.end_station,
    t.start_time,
    bc.chain_position + 1
  FROM trips t
  JOIN bike_chain bc
    ON t.bike_id = bc.bike_id
    AND t.start_station = bc.end_station
    AND t.start_time > bc.start_time
)
SELECT * FROM bike_chain ORDER BY chain_position;
```

The anchor finds that bike's first trip of the day. Each recursive step looks for the *next* trip whose start station matches the previous trip's end station — reconstructing the bike's full path across however many trips it took, without knowing in advance how many links the chain has. A fixed number of self-joins couldn't do this, because you don't know ahead of time how many trips are in the chain; recursion handles an unknown depth naturally.

## A Guardrail: Recursive CTEs Can Loop Forever

Unlike everything else this week, a recursive CTE with a mistake in its terminating condition doesn't just give a wrong answer — it can genuinely run forever, or until it exhausts memory. The `WHERE calendar_date < DATE '2025-12-31'` in the date spine example isn't optional detail; it's the thing that stops the recursion. Most databases also support a safety valve:

```sql
WITH RECURSIVE date_spine AS (
  ...
)
SELECT calendar_date FROM date_spine
LIMIT 10000;
```

A `LIMIT` doesn't fix a broken terminating condition, but it turns "the query hangs indefinitely" into "the query returns a suspiciously round number of rows," which is a much easier bug to notice.

---

## Query Performance Basics: Reading `EXPLAIN`

Every query you've written this course has produced a result. `EXPLAIN` shows you the database's **plan** for producing that result — which tables it reads, in what order, using what strategy — without actually running the query for real (some databases offer `EXPLAIN ANALYZE`, which does run it and reports actual timing alongside the plan).

```sql
EXPLAIN
SELECT station_name, COUNT(*) AS trip_count
FROM trips
WHERE start_station = 'Pentagon City'
GROUP BY station_name;
```

Reading a plan doesn't require memorizing every operator your specific engine uses, but two ideas cover most of what you need this week:

**Sequential scan vs. index scan.** A sequential (or "seq" / "table") scan reads every row in a table to find the ones that match. An index scan uses a separate structure to jump directly to matching rows, skipping the rest. For a small table, a sequential scan is often *faster* than an index scan, because the overhead of consulting the index outweighs just reading everything — this is why a query planner will sometimes deliberately ignore an available index, which can look wrong at first glance but usually isn't.

**Estimated rows vs. actual rows.** A plan shows how many rows the planner *expects* each step to produce. When you have `EXPLAIN ANALYZE` available, comparing the estimate against the actual count is one of the most useful diagnostic habits in SQL: a huge mismatch usually means the planner's statistics about your data are stale or wrong, which can cause it to pick a genuinely bad plan even though the query is logically correct.

## Indexing, Conceptually

An index is a separate, ordered structure built on one or more columns, letting the database find matching rows without scanning the whole table — similar to how a book's index lets you find a topic without reading every page. The trade-off is real: an index speeds up lookups and filters on the indexed column, but it costs storage and slows down writes, since every `INSERT` or `UPDATE` has to keep the index current too. Indexes tend to help most on columns you frequently filter (`WHERE`), join (`ON`), or sort (`ORDER BY`) by — and matter far less on columns you rarely constrain, or on tables small enough that a full scan is already fast.

One caveat worth knowing if you're working in DuckDB, as this course's exercises assume: DuckDB is a **columnar, vectorized** engine, which changes the calculus around indexing quite a bit compared to a traditional row-store database like PostgreSQL or MySQL. DuckDB can scan a single column extremely fast without a traditional index at all, because it only reads the columns a query actually references, in large vectorized batches — so you'll see far less emphasis on manually creating indexes in DuckDB than you would in a transactional database tuned for many small lookups. The *concepts* — scan vs. targeted access, the cost of maintaining a structure on write — carry over regardless of engine; the specific tuning advice doesn't always.

---

## Pivoting: Rows Into Columns

A pivot reshapes long data (many rows, few columns) into wide data (fewer rows, more columns) — usually turning the distinct values of one column into column headers of their own. DuckDB supports this directly with a `PIVOT` statement:

```sql
PIVOT (
  SELECT rider_type, DATE_TRUNC('month', start_time) AS ride_month, COUNT(*) AS trip_count
  FROM trips
  GROUP BY rider_type, ride_month
)
ON rider_type
USING SUM(trip_count)
GROUP BY ride_month;
```

This turns one row per `(rider_type, ride_month)` pair into one row per `ride_month`, with a separate column for each `rider_type`'s trip count — the shape you'd actually want for a report or a chart with one line per rider type. In engines without native `PIVOT` syntax, the same result is built by hand with conditional aggregation:

```sql
SELECT
  DATE_TRUNC('month', start_time) AS ride_month,
  SUM(CASE WHEN rider_type = 'member' THEN 1 ELSE 0 END) AS member_trips,
  SUM(CASE WHEN rider_type = 'casual' THEN 1 ELSE 0 END) AS casual_trips
FROM trips
GROUP BY ride_month;
```

Both produce the same shape. The manual version is worth knowing even where native `PIVOT` exists, since it's the version that works everywhere and makes explicit what's actually happening: one `SUM` per resulting column, each filtered to a different value.

## Unpivoting: Columns Back Into Rows

Unpivoting is the reverse: wide data back into long data, turning several columns into rows with a label column identifying which original column each value came from. DuckDB's `UNPIVOT`:

```sql
UNPIVOT monthly_wide_table
ON member_trips, casual_trips
INTO
  NAME rider_type
  VALUE trip_count;
```

This is genuinely common as a *first* step, not just a report-formatting trick: raw source data — especially anything originally exported from a spreadsheet, like a wide table with one column per month — usually needs to be unpivoted into a proper long format before window functions, `GROUP BY`, or joins can be applied to it cleanly. Long format is the shape almost everything else this course has assumed; pivot is what you build at the very end, for the person reading the final report.

---

## Looking Ahead: Where This Fits Once dbt Enters the Picture

Today's three topics map onto dbt in three different ways worth previewing now. The date-spine recursive CTE is a strong candidate for its own small model — something like `dim_date_spine` — that other models simply reference by name instead of rebuilding. Reading `EXPLAIN` plans doesn't change inside dbt, but it becomes more important: a dbt project can chain dozens of models together, and a slow one buried in the middle of that chain is a lot harder to spot than a single slow query sitting on its own. And the pivot/unpivot pattern shows up constantly in the layered structure dbt encourages — raw sources arrive wide or messy, get unpivoted into long "staging" models, get transformed, and then sometimes get pivoted back into wide "mart" models at the very end for whatever's consuming the final output.

## What's Next

Today's task file builds a working date-spine recursive CTE, follows a bike across a multi-trip chain, reads real `EXPLAIN` output on this course's dataset, and builds one pivoted and one unpivoted report from the same underlying data. Day 4 sets all of this aside and asks a different question entirely: not *how* to write a query, but how a real data team organizes dozens of queries like these — written this week and in the weeks before it — into something they can actually trust, test, and rerun. That's what dbt is for.