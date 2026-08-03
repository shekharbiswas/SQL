# Day 2 Concepts: Window Frames, Running Totals, Moving Averages & Volatility

**Estimated reading time: 55 min**

Day 1 introduced `OVER()` and `PARTITION BY`, which control *which group* of rows a window function looks at. Today introduces the other half of that control: the **frame** — exactly *which rows within the ordering* a function should consider, relative to the current row. This is what makes running totals and moving averages possible, and it's also where Week 2, Day 4's correlated-subquery trailing average is replaced with a window function version, rather than just previewed.

Today also introduces a second dataset not yet used in this course: daily **weather** observations, matched to trip activity by date and by the weather station nearest each bike station's event location. Everything covered so far has measured the *center* of a set of rows — an average, a total. The weather section measures the *spread* — how volatile conditions were — using the same `OVER()` mechanism.

---

## What a Frame Actually Is

When you write `OVER (ORDER BY calendar_date)`, you're not just saying "in this order" — you're implicitly defining a frame too, even if you don't write one explicitly. The frame is the specific slice of rows, relative to the current row, that the function actually computes over.

The full frame syntax looks like this:

```sql
SUM(total_rides) OVER (
  ORDER BY calendar_date
  ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
)
```

`ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` means: for each row, look at the 6 rows before it plus itself — 7 rows total. The frame slides as you move down the result set, which is exactly what a trailing 7-day window means.

## The Frame Boundaries

Each side of `BETWEEN ... AND ...` can be one of:

- `UNBOUNDED PRECEDING` — start from the very first row in the partition
- `N PRECEDING` — start N rows before the current row
- `CURRENT ROW` — the row being computed
- `N FOLLOWING` — N rows after the current row
- `UNBOUNDED FOLLOWING` — go all the way to the last row in the partition

Mixing these gives you every practical pattern you'll need:

```sql
-- Running total from the beginning through the current row
SUM(total_rides) OVER (ORDER BY calendar_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

-- Trailing 7-row average (today plus the 6 before it)
AVG(total_rides) OVER (ORDER BY calendar_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)

-- Centered 7-row average (3 before, today, 3 after)
AVG(total_rides) OVER (ORDER BY calendar_date ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING)

-- Everything in the partition, regardless of current row (same as no ORDER BY frame at all)
AVG(total_rides) OVER (ORDER BY calendar_date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
```

## The Default Frame — Why Running Totals "Just Work"

Here's the part that catches people off guard. If you write `OVER (ORDER BY calendar_date)` with **no explicit frame at all**, most databases apply a default frame of `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` — which is exactly a running total's frame. That's why this works without you ever writing `ROWS BETWEEN`:

```sql
SELECT
  calendar_date,
  total_rides,
  SUM(total_rides) OVER (ORDER BY calendar_date) AS running_total
FROM daily_series
ORDER BY calendar_date;
```

This is convenient for running totals specifically, but it's also a trap for anything else. If you write `AVG(total_rides) OVER (ORDER BY calendar_date)` expecting an overall average, you won't get one — you'll get a running average through the current row, because the same default frame applies. If you want the average across the *entire* partition regardless of row position, you need either no `ORDER BY` at all, or an explicit `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`. The same trap applies to `STDDEV()` later in this file — worth remembering before you get there.

## `ROWS` vs. `RANGE`: A Real Difference, Not Just Syntax

`ROWS` counts physical rows — "6 rows before this one," full stop. `RANGE` counts by **value**, treating rows with the same `ORDER BY` value as tied peers that all get included together, regardless of how many physical rows that is.

```sql
-- ROWS: exactly the 2 rows before this one, by position
SUM(total_rides) OVER (ORDER BY calendar_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)

-- RANGE: every row with a date value within the range, which matters when values repeat
SUM(total_rides) OVER (ORDER BY calendar_date RANGE BETWEEN INTERVAL '2 days' PRECEDING AND CURRENT ROW)
```

For a daily series with one row per unique date, `ROWS` and `RANGE` often behave identically, since there are no ties to group. The difference becomes real once your `ORDER BY` column has duplicate values — ranking trips by `start_time` truncated to the hour, for instance, where several trips can share the same hour. `RANGE` would treat all same-hour rows as one tied group; `ROWS` would still count them individually by physical position. Defaulting to `ROWS` when you mean "N rows back" and reaching for `RANGE` specifically when you mean "N units of value back" avoids a subtle mismatch between what you wrote and what you meant.

## Running Totals

The pattern from earlier, generalized: a running total answers "what's the cumulative sum up through this point," which is naturally suited to `SUM() OVER (ORDER BY ... )` with the default frame.

```sql
SELECT
  calendar_date,
  total_rides,
  SUM(total_rides) OVER (ORDER BY calendar_date) AS cumulative_rides
FROM daily_series
ORDER BY calendar_date;
```

Running totals partition naturally too — a running total *per station*, for example, resets at each new station:

```sql
SELECT
  station_name,
  calendar_date,
  daily_rides,
  SUM(daily_rides) OVER (PARTITION BY station_name ORDER BY calendar_date) AS cumulative_rides_at_station
FROM station_daily_series
ORDER BY station_name, calendar_date;
```

## Moving (Trailing) Averages

This is the direct replacement for Week 2, Day 4's correlated subquery:

```sql
SELECT
  calendar_date,
  total_rides,
  AVG(total_rides) OVER (
    ORDER BY calendar_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS trailing_7day_avg
FROM daily_series
ORDER BY calendar_date;
```

Notice what's gone compared to the Week 2 version: no correlated subquery, no re-scanning per row from the database's perspective in the same way, and no separate `BETWEEN` date-arithmetic condition to get right. The frame clause says exactly what you mean, once.

## Period-over-Period Percent Change

Combining `LAG()` from Day 1 with basic arithmetic gets you from a raw comparison to a percentage, which is usually the more useful number in a real report:

```sql
SELECT
  calendar_date,
  total_rides,
  LAG(total_rides) OVER (ORDER BY calendar_date) AS previous_day_rides,
  ROUND(
    100.0 * (total_rides - LAG(total_rides) OVER (ORDER BY calendar_date))
    / NULLIF(LAG(total_rides) OVER (ORDER BY calendar_date), 0),
    1
  ) AS pct_change
FROM daily_series
ORDER BY calendar_date;
```

`NULLIF(..., 0)` guards against a division-by-zero error on any day where the previous day had exactly zero rides — worth keeping in mind given Week 2, Day 2's gap-detection work turned some genuinely-zero days into visible `0`s rather than `NULL`s.

---

## Measuring Spread, Not Just Center: `STDDEV()` as a Window Function

Everything above has answered "what's the typical value near this row." `STDDEV()` answers a different question: "how much did nearby values *vary*?" It works exactly like `AVG()` — same `OVER()` clause, same `PARTITION BY`, same frame syntax — but returns a standard deviation instead of a mean.

```sql
SELECT
  weather_date,
  temp_high,
  STDDEV(temp_high) OVER (
    ORDER BY weather_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS trailing_7day_temp_stddev
FROM daily_weather
ORDER BY weather_date;
```

A low trailing standard deviation means temperature has been stable over that week; a high one means it's been swinging — a genuinely different signal from the trailing average, which could stay flat even while conditions swing wildly day to day.

Most engines expose two variants: `STDDEV_SAMP()` (sample standard deviation, the usual default and what plain `STDDEV()` maps to) and `STDDEV_POP()` (population standard deviation, dividing by `N` instead of `N - 1`). For a single weather station's full observed history, you're working with the complete population of days you have, not a sample meant to generalize to some larger population — so `STDDEV_POP()` is arguably the more defensible choice here, though in practice the two converge closely once a window has more than a handful of rows. `VARIANCE()` (or `VAR_SAMP()` / `VAR_POP()`) is the same idea one step earlier — standard deviation is just the square root of variance — and follows the identical `OVER()` pattern if you ever need it.

## Joining Trips to Weather by Event Location

The `daily_weather` table holds one row per date per weather station: `weather_date`, `weather_station_id`, `temp_high`, `temp_low`, `precip_in`, `wind_mph`. Each bike station has a `nearest_weather_station_id` column, matching it to the weather station physically closest to it — meaning a trip's start location determines which weather observation actually applies to it, not just the date. Joining the two means matching on *both* the date and the location, not date alone:

```sql
SELECT
  t.station_name,
  t.calendar_date,
  w.temp_high,
  w.precip_in
FROM station_daily_series t
JOIN stations s ON s.station_name = t.station_name
JOIN daily_weather w
  ON w.weather_station_id = s.nearest_weather_station_id
  AND w.weather_date = t.calendar_date
ORDER BY t.station_name, t.calendar_date;
```

This is the setup for today's weather-volatility block: once trips and weather are joined at the event-location level, `STDDEV() OVER (...)` on the weather columns, partitioned by station and ordered by date, gives you a genuine "how unstable was the weather near this station recently" signal — one you can then compare against ride-count volatility at the same station over the same window.

---

## Looking Ahead: Where This Fits Once dbt Enters the Picture

The station-to-weather-station join above is a good example of something that gets awkward to repeat by hand: every query that needs weather-adjusted trip data has to re-join `stations`, `daily_weather`, and whatever trip table it starts from, the same three-table join, over and over. In a dbt project, you'd write that join exactly once as its own model — something like `stg_trips_with_weather` — and every later query, including the `STDDEV()` work below, would simply select from that model instead of repeating the join. That's most of what "organizing SQL into a pipeline" actually means in practice: naming a repeated piece of logic once, instead of retyping it in every query that needs it. Day 4 picks this up directly using this exact join as the first real example.

## What's Next

Today's task file builds running totals, trailing and centered moving averages, percent-change comparisons, and trailing standard deviation of weather against the daily series and the new weather-joined dataset, then folds the moving-average work directly back into Week 2, Day 4's anomaly-detection pipeline — replacing the correlated subquery there for good. Day 3 moves into a set of tools that don't fit the "compare this row to nearby rows" theme at all: recursive CTEs for self-referencing data, `EXPLAIN` plans and indexing for understanding *why* a query is slow, and pivoting/unpivoting for reshaping rows into columns and back.