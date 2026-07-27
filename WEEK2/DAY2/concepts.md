# Day 2: Common Table Expressions (CTEs)

**Estimated reading time: 45 min**

Day 1 ended with a real complaint, if you wrote out the Block 5 reflection honestly: subqueries nested inside subqueries get hard to track. A `FROM` subquery containing a `WHERE` subquery reads inside-out — you have to find the innermost query, understand it first, then work your way back out to understand what the outer query does with it. That's backwards from how most people actually think through a problem, which is top to bottom, step by step.

CTEs fix exactly that. Nothing in this file makes subqueries obsolete — you'll still reach for a scalar subquery in `WHERE` all the time — but for anything with more than one logical step, a CTE is almost always the more readable choice.

---

## What a CTE Actually Is

A CTE is a named, temporary result set that exists only for the duration of one query. You define it with `WITH`, give it a name, and then reference that name later in the query as if it were a real table.

```sql
WITH high_capacity_stations AS (
  SELECT station_name, capacity, neighborhood
  FROM stations
  WHERE capacity > 30
)
SELECT neighborhood, COUNT(*) AS station_count
FROM high_capacity_stations
GROUP BY neighborhood;
```

Compare this to the equivalent written as a `FROM` subquery:

```sql
SELECT neighborhood, COUNT(*) AS station_count
FROM (
  SELECT station_name, capacity, neighborhood
  FROM stations
  WHERE capacity > 30
) AS high_capacity_stations
GROUP BY neighborhood;
```

Same result, same execution, but the CTE version reads in the order you'd actually explain it out loud: "first, here are the high-capacity stations — now, count them by neighborhood." The subquery version buries that first step in the middle of the `FROM` clause.

## Multiple CTEs in One Query

You're not limited to one. Separate multiple CTEs with commas, and each one can be queried independently in the final `SELECT` — or joined together.

```sql
WITH station_totals AS (
  SELECT start_station, COUNT(*) AS total_rides
  FROM trips
  GROUP BY start_station
),
high_capacity AS (
  SELECT station_name FROM stations WHERE capacity > 30
)
SELECT st.start_station, st.total_rides
FROM station_totals st
JOIN high_capacity hc ON st.start_station = hc.station_name
ORDER BY st.total_rides DESC;
```

Each CTE here answers one clear sub-question — ride totals per station, and which stations are high capacity — and the final `SELECT` combines them. If you needed to debug this, you could run `station_totals` or `high_capacity` on their own, independent of the rest of the query, just by selecting from them directly. That's much harder to do with a nested subquery, where the inner logic isn't a standalone, nameable thing.

## Chaining CTEs

A CTE can reference an earlier CTE defined in the same `WITH` clause. This is how you build a multi-step transformation that reads as a sequence of named stages, each one building on the last.

```sql
WITH station_totals AS (
  SELECT start_station, COUNT(*) AS total_rides
  FROM trips
  GROUP BY start_station
),
station_with_neighborhood AS (
  SELECT st.start_station, st.total_rides, s.neighborhood
  FROM station_totals st
  JOIN stations s ON st.start_station = s.station_name
),
neighborhood_averages AS (
  SELECT neighborhood, AVG(total_rides) AS avg_rides
  FROM station_with_neighborhood
  GROUP BY neighborhood
)
SELECT swn.start_station, swn.total_rides, swn.neighborhood, na.avg_rides
FROM station_with_neighborhood swn
JOIN neighborhood_averages na ON swn.neighborhood = na.neighborhood
WHERE swn.total_rides > na.avg_rides;
```

Three stages, each with a name, each doing one thing: totals, then attach neighborhood, then average by neighborhood. The final query just combines the last two. This is the same kind of "aggregate of an aggregate" problem from Day 1's `FROM` subquery block — but chained CTEs make each step visible and testable on its own, instead of collapsing it all into one nested block.

## CTE vs. Subquery vs. Materialized Table

These three can look similar from a distance, but they behave differently:

- A **subquery** is evaluated inline, as part of the larger query, every time.
- A **CTE** is also evaluated as part of the query execution — it is *not* automatically saved or cached. Depending on the database engine, it may be computed once and reused, or re-computed each time it's referenced. Don't assume a CTE is free just because it has a name; assume it costs roughly what the equivalent subquery would.
- A **materialized table** — like `stations_summary` from Week 1 — is actually computed once and stored. Querying it later doesn't re-run any of the original logic.

The choice isn't about performance first. It's about matching the tool to whether the result needs to persist (materialized table), needs to be reused within a single query (CTE), or is a one-off filter or calculation (subquery).

## A Preview: Recursive CTEs

Everything above is a **non-recursive** CTE — it's defined once and used as-is. There's also a **recursive** form, where a CTE can reference itself to walk through hierarchical or sequential data (an org chart, a chain of station relocations, a running sequence). That's a Week 3 topic, not today's — it depends on being comfortable with ordinary CTEs first, which is what today's task file is for.

---

## What's Next

Today's task file asks you to rewrite some of Day 1's subqueries as CTEs, then go further than Day 1 could reasonably go — multi-step, chained transformations that would have been unreadable as nested subqueries. From there, Day 3 moves into `NULL` handling, `COALESCE`, and `CASE WHEN` — the tools for actually cleaning the inconsistencies you've been working around since Week 1.