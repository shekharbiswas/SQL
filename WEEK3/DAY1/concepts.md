# Day 1 Concepts: Window Function Fundamentals

**Estimated reading time: 50 min**

Week 2, Day 4 solved trailing averages using a correlated subquery, and period-over-period comparison using a self-join. Both produced correct results, but the correlated subquery re-scanned a window of rows for every single row in the table, and the self-join required more machinery than the question — "how does this row compare to nearby rows" — actually needed. Window functions are the tool built specifically to answer that kind of question, without a self-join and without a correlated subquery re-running per row.

This is the first of several tools this week that come up constantly in production analytics SQL. Today is about understanding what window functions actually do, not just memorizing the syntax.

This week also ends somewhere new: Day 4 and Day 5 introduce **dbt**, the tool most data teams actually use to organize SQL like the queries you're about to write into a maintainable pipeline. Nothing about dbt needs to make sense yet — but starting today, watch for a short callout at the end of each concepts file connecting what you just learned to how it will eventually live inside a dbt project. By Day 4, none of it should feel like it came out of nowhere.

---

## The Core Idea: A Window Function Doesn't Collapse Rows

`GROUP BY` and aggregate functions collapse many rows into one — ten rows of trip data at a station become one row with a total count. That's exactly what you want sometimes, and exactly not what you want other times. If you want to know each individual trip's rank *within* its station, `GROUP BY` can't give you that — it destroys the individual rows on the way to the aggregate.

A window function computes something aggregate-like — a rank, a running total, a comparison to another row — but keeps every original row intact. You get the individual row **and** something calculated across a set of related rows, side by side.

```sql
SELECT
  station_name,
  capacity,
  RANK() OVER (ORDER BY capacity DESC) AS capacity_rank
FROM stations;
```

Every station still appears as its own row. Nothing got collapsed. `RANK()` just adds a new column computed by looking across the whole result set.

## The `OVER()` Clause

`OVER()` is what makes a function a window function instead of a regular aggregate. It defines the **window** — the set of rows the function should consider for each row's calculation. An empty `OVER()` means "the entire result set is the window for every row."

```sql
SELECT
  station_name,
  capacity,
  AVG(capacity) OVER () AS overall_avg_capacity
FROM stations;
```

This looks similar to Day 1 Week 2's scalar subquery trick for attaching an average to every row — and it produces the same result — but it's a single window function call instead of a separate nested query. That's the pattern you'll see repeatedly this week: something Week 1–2 solved with a subquery, join, or self-join often has a shorter, clearer window function equivalent.

## `PARTITION BY`: Windows Within Groups

Most of the time you don't want the window to be the entire table — you want it scoped per group, similar to what `GROUP BY` does, but without collapsing rows. `PARTITION BY` does exactly that.

```sql
SELECT
  station_name,
  neighborhood,
  capacity,
  AVG(capacity) OVER (PARTITION BY neighborhood) AS neighborhood_avg_capacity
FROM stations;
```

Now each row gets the average capacity *for its own neighborhood*, not the whole table — while every individual station row is still present. This is the direct window-function replacement for the correlated subquery pattern from Week 1, Day 1, Block 4 ("stations above their neighborhood's average"), and for the chained-CTE version from Week 2, Day 2.

## `ROW_NUMBER()`: A Unique, Arbitrary Rank

`ROW_NUMBER()` assigns a strictly increasing integer to each row within its window, based on the `ORDER BY` inside `OVER()`. Ties get broken arbitrarily — two rows with the same value still get two different row numbers.

```sql
SELECT
  station_name,
  capacity,
  ROW_NUMBER() OVER (ORDER BY capacity DESC) AS row_num
FROM stations;
```

This is often used to pull "the top N per group" — a genuinely common real-world need:

```sql
SELECT *
FROM (
  SELECT
    station_name,
    neighborhood,
    total_rides,
    ROW_NUMBER() OVER (PARTITION BY neighborhood ORDER BY total_rides DESC) AS rn
  FROM stations_summary
) ranked
WHERE rn = 1;
```

This finds the single busiest station *per neighborhood* — the same question as Week 2, Day 2's Block 4 mini-project, previously solved with three chained CTEs. Here it's one window function and a filter.

## `RANK()` and `DENSE_RANK()`: Ranking With Ties

`ROW_NUMBER()` never ties. `RANK()` and `DENSE_RANK()` both do, but they handle what happens *after* a tie differently.

```sql
SELECT
  station_name,
  capacity,
  ROW_NUMBER() OVER (ORDER BY capacity DESC) AS row_num,
  RANK() OVER (ORDER BY capacity DESC) AS rank_num,
  DENSE_RANK() OVER (ORDER BY capacity DESC) AS dense_rank_num
FROM stations;
```

If two stations tie for the highest capacity:
- `ROW_NUMBER()` gives them `1` and `2` — an arbitrary, forced distinction.
- `RANK()` gives them both `1`, then **skips** `2` entirely — the next distinct value gets `3`.
- `DENSE_RANK()` gives them both `1`, and the next distinct value gets `2` — no gap.

Which one is "correct" depends on the question. If you're building a leaderboard where skipped positions make sense (two people tied for 1st means there's no 2nd place), `RANK()` matches that intuition. If you want every distinct value to get a consecutive rank with no gaps, `DENSE_RANK()` is usually what you actually want.

## `LAG()` and `LEAD()`: Looking at Neighboring Rows

`LAG()` pulls a value from a **previous** row in the window's order; `LEAD()` pulls one from a **following** row. This directly replaces the self-join pattern from Week 2, Day 4.

```sql
SELECT
  calendar_date,
  total_rides,
  LAG(total_rides) OVER (ORDER BY calendar_date) AS previous_day_rides,
  total_rides - LAG(total_rides) OVER (ORDER BY calendar_date) AS day_over_day_change
FROM daily_series
ORDER BY calendar_date;
```

Compare this to the self-join from Week 2, Day 4 — same result, no join at all. `LAG()` and `LEAD()` both accept an optional second argument for how many rows back or forward to look (default is 1), and an optional third argument for what to return when there's no such row (default is `NULL`):

```sql
LAG(total_rides, 7, 0) OVER (ORDER BY calendar_date) AS rides_one_week_ago
```

This looks back 7 rows instead of 1, and returns `0` instead of `NULL` if there's no row that far back (for example, near the very start of the dataset).

## `ORDER BY` Inside `OVER()` vs. Outside It

These are easy to conflate but do different jobs. The `ORDER BY` **inside** `OVER(...)` defines the order the window function itself uses — which row counts as "previous" for `LAG`, or what order ranks are assigned in. The `ORDER BY` **outside**, at the end of the whole query, controls the order the final result set is displayed in. You often want both, and they don't have to match:

```sql
SELECT
  station_name,
  total_rides,
  RANK() OVER (ORDER BY total_rides DESC) AS ridership_rank
FROM stations_summary
ORDER BY station_name;
```

Here, ranks are assigned by ridership (highest first), but the final output is displayed alphabetically by station name — the rank numbers won't appear in order down the page, which is intentional and worth expecting rather than being confused by.

## Almost Any Aggregate Can Become a Window Function

Everything above used `AVG`, `SUM`, `RANK`, `ROW_NUMBER`, `LAG`, and `LEAD`, but the pattern is general: nearly any aggregate function — `COUNT`, `MIN`, `MAX`, `SUM`, `AVG`, and also `STDDEV` / `VARIANCE` — becomes a window function the moment you add `OVER(...)` after it. Tomorrow uses this directly: `STDDEV(...) OVER (...)` measures how *volatile* a set of nearby rows is, the same way `AVG(...) OVER (...)` measures their center. Same clause, same mental model, different question.

---

## Looking Ahead: Where This Fits Once dbt Enters the Picture

You don't need this to make sense yet, but keep it in the back of your mind: the SQL you're writing today — a `SELECT` with a window function in it — is exactly the kind of statement that becomes a **dbt model** starting Day 4. A dbt project is, at its core, a folder of `.sql` files very close to what you're writing this week, each one a single `SELECT`, wired together so dbt knows which ones depend on which. The window-function queries you're building today for one-off analysis are the same shape as the queries that will later live inside `models/marts/` or `models/staging/` in a real dbt project. Nothing changes about how you write the SQL — what changes is how it gets organized, tested, and run.

## What's Next

Today's task file is entirely about getting comfortable with `OVER()`, `PARTITION BY`, and these five functions on their own. Day 2 builds directly on this — running totals, moving averages, and period-over-period comparisons using **window frames**, which extend `OVER()` to define not just an order but a specific range of rows relative to the current row — and introduces a new weather dataset to measure volatility, not just averages.