# Day 1: Understanding Subqueries

**Estimated reading time: 45 min**

Week 1 ended with `stations_summary` — a table built by joining and aggregating in one pass. That works when the question you're answering is simple enough to express in a single `SELECT`. A lot of real questions aren't. "Which stations are above average capacity?" needs an average computed first, then compared against. "Which rides happened at the five busiest stations?" needs a ranking computed first, then filtered against. Subqueries are how SQL answers a question that depends on the answer to a smaller question.

This file explains the concept before Day 1's task file asks you to write any. Read it first — the task file assumes you already know the vocabulary used here.

---

## What a Subquery Actually Is

A subquery is a `SELECT` statement nested inside another SQL statement. The inner query runs (conceptually) first, and its result becomes an input to the outer query — a value to compare against, a set to check membership in, or a whole table to query further.

```sql
SELECT station_name, capacity
FROM stations
WHERE capacity > (SELECT AVG(capacity) FROM stations);
```

Here, `(SELECT AVG(capacity) FROM stations)` is the subquery. It produces one number — the average capacity across all stations — and the outer query uses that number as a comparison value. Nothing about this requires a join, because you're not combining two tables' columns; you're using one table's aggregate to filter that same table.

## Three Places a Subquery Can Live

**In `WHERE`** — filtering rows based on something computed separately:

```sql
WHERE capacity > (SELECT AVG(capacity) FROM stations)
```

**In `FROM`** — treating a query's result as if it were a table:

```sql
SELECT neighborhood, AVG(total_rides)
FROM (
  SELECT neighborhood, station_name, COUNT(*) AS total_rides
  FROM trips t
  JOIN stations s ON t.start_station = s.station_name
  GROUP BY neighborhood, station_name
) AS station_totals
GROUP BY neighborhood;
```

This is useful when you need to aggregate something that is *itself* already an aggregate — average ride count per station, then averaged again per neighborhood. You can't do that in one `GROUP BY`.

**In `SELECT`** — pulling a single computed value into each row of the outer query:

```sql
SELECT
  station_name,
  capacity,
  (SELECT AVG(capacity) FROM stations) AS avg_capacity
FROM stations;
```

Every row gets the same average value attached to it, which is handy for row-by-row comparisons without a separate query.

## Scalar vs. Multi-Row Subqueries

A **scalar subquery** returns exactly one value — one row, one column. `(SELECT AVG(capacity) FROM stations)` is scalar. Scalar subqueries can be used almost anywhere a literal value could go: `WHERE capacity > (...)`, `SELECT (...) AS avg_capacity`.

A **multi-row subquery** returns a list of values. You can't compare a column to a list with `=`; you need `IN`, `ANY`, or `ALL` instead:

```sql
SELECT *
FROM trips
WHERE start_station IN (
  SELECT station_name FROM stations WHERE capacity > 30
);
```

This says: give me every trip that started at a station in this list of high-capacity stations. If you tried `= (SELECT ...)` here instead of `IN`, it would fail as soon as the subquery returned more than one row.

## `EXISTS` — a Different Kind of Question

`IN` asks "is this value in that list?" `EXISTS` asks a different question: "does at least one matching row exist?" It doesn't care about the actual values returned by the subquery — only whether any row comes back at all.

```sql
SELECT station_name
FROM stations s
WHERE EXISTS (
  SELECT 1 FROM trips t WHERE t.start_station = s.station_name
);
```

This returns every station that has ever been used as a start station — without needing to join `trips` and then deduplicate. `EXISTS` is often faster than `IN` on large tables, because the database can stop looking as soon as it finds one match, instead of building the full list first.

## Correlated Subqueries

Every example so far except the `EXISTS` one has been **non-correlated** — the subquery is completely self-contained and could run on its own, independent of the outer query.

A **correlated subquery** references a column from the outer query inside the subquery itself, which means it can't run standalone — it has to run once per row of the outer query, using that row's value each time. The `EXISTS` example above is correlated: `t.start_station = s.station_name` reaches out to `s`, which only exists because of the outer query's `FROM stations s`.

Correlated subqueries are more expensive — conceptually, the database re-runs the inner query for every outer row — but they answer questions a non-correlated subquery can't, like "for each station, how does its ride count compare to other stations in the same neighborhood?"

```sql
SELECT s.station_name, s.neighborhood,
  (SELECT COUNT(*) FROM trips t WHERE t.start_station = s.station_name) AS ride_count
FROM stations s;
```

## Subquery vs. Join: When to Use Which

These often overlap — several examples above could be rewritten as joins. A rough guide:

- If you need **columns from both tables** in the final result, you need a join.
- If you only need to **filter or compare** based on another table, without pulling its columns into the output, a subquery is often clearer.
- If the question involves **an aggregate of an aggregate** (average per group, then averaged again), a subquery in `FROM` is usually the only clean way to express it.
- If you're checking **existence** rather than pulling values, `EXISTS` is typically both clearer and faster than the equivalent join-and-deduplicate.

Neither is universally "better." Readability for the specific question usually decides it.

## The Pitfall Worth Knowing Now: `NOT IN` and `NULL`

This one causes real bugs. If the subquery in a `NOT IN` can return a `NULL`, the entire condition can silently return no rows at all — not an error, just an empty result that looks like "nothing matched" when actually SQL couldn't determine a match either way.

```sql
-- Dangerous if start_station can be NULL anywhere in trips
SELECT * FROM stations
WHERE station_name NOT IN (SELECT start_station FROM trips);
```

If even one row in `trips` has a `NULL` `start_station`, this query can return zero rows, regardless of what the actual data looks like. `NOT EXISTS` with a correlated subquery doesn't have this problem, which is one reason `EXISTS`/`NOT EXISTS` are often preferred over `IN`/`NOT IN` once `NULL`s are in play — and given Week 1's join notes, you already know `NULL`s show up in this dataset.

---

## What's Next

CTEs — introduced later this week — solve a different problem: subqueries nested inside subqueries get hard to read fast. A CTE lets you name an intermediate result and build on it step by step, top to bottom, instead of reading queries inside-out. Today's task file sticks to subqueries on their own so that comparison actually means something once you get there.