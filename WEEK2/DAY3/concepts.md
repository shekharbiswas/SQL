# Day 3 Concepts: NULL Handling, COALESCE & CASE WHEN

**Estimated reading time: 40 min**

Since Week 1, Day 4, you've had a documented problem sitting in `notes/data_quality.md`: some stations referenced in `trips` don't match anything in `stations`, which means certain joins produce `NULL`s for capacity and neighborhood. You've worked around it — filtering it out, noting it, deciding case by case whether to include it. Today gives you the actual tools to handle it deliberately instead of working around it: what `NULL` really means in SQL, how to supply defaults with `COALESCE`, and how to build your own categories with `CASE WHEN`.

---

## What `NULL` Actually Means

`NULL` isn't a value — it's the *absence* of one. It doesn't mean zero, it doesn't mean an empty string, and it doesn't behave like either in comparisons. This is the part that trips people up: `NULL` doesn't equal anything, including itself.

```sql
SELECT NULL = NULL;   -- returns NULL, not TRUE
SELECT NULL = 5;      -- returns NULL, not FALSE
SELECT NULL <> 5;     -- also returns NULL, not TRUE
```

SQL uses **three-valued logic**: every comparison is `TRUE`, `FALSE`, or `NULL` (meaning "unknown"). A `WHERE` clause only keeps rows where the condition evaluates to `TRUE` — rows that evaluate to `NULL` are silently dropped, not treated as a match or a non-match. This is exactly what caused the `NOT IN` pitfall from Day 1: if the list being checked against contains a `NULL`, every comparison against that list can come back `NULL` instead of `TRUE` or `FALSE`, and the whole condition quietly fails.

To explicitly test for `NULL`, you need dedicated syntax, not `=`:

```sql
SELECT * FROM stations WHERE neighborhood IS NULL;
SELECT * FROM stations WHERE neighborhood IS NOT NULL;
```

## Aggregate Functions and `NULL`

Most aggregate functions quietly ignore `NULL`s rather than erroring or treating them as zero. `AVG(duration)` divides by the count of non-`NULL` rows, not the total row count. `COUNT(column)` counts non-`NULL` values in that column, while `COUNT(*)` counts every row regardless. These can legitimately give different numbers on the same table, and it's worth checking which one you actually want:

```sql
SELECT COUNT(*) AS total_rows, COUNT(duration) AS rows_with_duration
FROM trips;
```

If those two numbers differ, some rides have a missing duration — worth knowing before you compute an average that silently excluded them.

## `COALESCE`: Supplying a Default

`COALESCE` takes a list of values and returns the first one that isn't `NULL`. Most often you'll use it with two arguments — a column and a fallback:

```sql
SELECT
  station_name,
  COALESCE(neighborhood, 'Unknown') AS neighborhood
FROM stations;
```

Any row where `neighborhood` is `NULL` now shows `'Unknown'` instead — which matters directly for a `GROUP BY neighborhood`, since grouping on a raw `NULL` column creates a group that's easy to overlook, while grouping on `COALESCE(neighborhood, 'Unknown')` makes that gap visible and countable.

`COALESCE` can chain more than two arguments, returning the first non-`NULL` in order:

```sql
SELECT COALESCE(actual_end_time, estimated_end_time, start_time) AS effective_end
FROM trips;
```

This tries `actual_end_time` first, falls back to `estimated_end_time` if that's missing, and falls back again to `start_time` only if both are missing.

## `NULLIF`: The Reverse Operation

`NULLIF(a, b)` returns `NULL` if `a` equals `b`, and returns `a` otherwise. It's useful for turning a placeholder value *into* a real `NULL`, which is often the first step before `COALESCE` can do anything useful:

```sql
SELECT NULLIF(neighborhood, 'N/A') AS neighborhood
FROM stations;
```

If some rows have the literal text `'N/A'` instead of an actual `NULL` — a common artifact of how CSV exports handle missing data — this converts it to a real `NULL`, so it behaves correctly in aggregates and `IS NULL` checks instead of being silently counted as a real neighborhood value.

## `CASE WHEN`: Building Your Own Categories

`CASE WHEN` is SQL's conditional expression — it lets you compute a value based on a series of conditions, evaluated top to bottom, stopping at the first match.

```sql
SELECT
  station_name,
  capacity,
  CASE
    WHEN capacity > 40 THEN 'Large'
    WHEN capacity > 20 THEN 'Medium'
    WHEN capacity > 0 THEN 'Small'
    ELSE 'Unknown'
  END AS capacity_tier
FROM stations;
```

`ELSE` catches anything that didn't match an earlier condition — including `NULL` capacity, since none of the `>` comparisons above would evaluate `TRUE` for a `NULL`. Leaving out `ELSE` doesn't cause an error; it just means unmatched rows get `NULL` in that column, which is worth doing deliberately rather than by accident.

`CASE WHEN` also works inside aggregates — a common pattern for turning row-level conditions into count-based summaries without a separate query per condition:

```sql
SELECT
  neighborhood,
  COUNT(CASE WHEN rider_type = 'casual' THEN 1 END) AS casual_rides,
  COUNT(CASE WHEN rider_type = 'member' THEN 1 END) AS member_rides
FROM trips t
JOIN stations s ON t.start_station = s.station_name
GROUP BY neighborhood;
```

Here, `CASE WHEN` returns `1` for matching rows and `NULL` otherwise (no `ELSE`), and since `COUNT` ignores `NULL`s, each `COUNT(CASE WHEN ...)` ends up counting only the rows matching that specific condition — effectively a conditional count, computed once per row instead of with separate filtered queries.

## Bringing It Together: Cleaning With Intent

The combination of `COALESCE` and `CASE WHEN` is how you turn "this data has gaps" into "here's exactly how I decided to handle each gap" — which is the difference between a clean-looking result and a genuinely defensible one. The unmatched-station problem from Week 1 isn't something today's tools make disappear; it's something today's tools let you handle on purpose, with the decision visible in the SQL itself instead of buried in a `WHERE` clause that quietly excluded it.

---

## What's Next

Today's task file applies `COALESCE` and `CASE WHEN` directly to the unmatched-station gap and to `rider_type` and duration inconsistencies you've likely noticed since Week 1. Day 4 moves into detecting anomalous periods in the trip data over time — separating a genuine trend from a data gap or a one-off event, which is exactly the kind of question that's only answerable once `NULL`s are handled deliberately rather than accidentally dropped.