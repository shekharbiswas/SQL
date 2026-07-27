# Day 4 Concepts: Detecting Anomalous Periods in a Time Series

**Estimated reading time: 55 min**

Everything this week has worked with `trips` as a set of rows to filter, join, and aggregate — but never really as a *sequence*. Today changes that. A seven-year, day-by-day view of ridership isn't just a bigger table; it's a timeline, and timelines have their own kinds of problems: gaps where data is simply missing, genuine shifts in behavior that persist, and one-off spikes or drops that don't mean anything on their own. Telling those apart is the actual skill this week has been building toward, and it's also the subject of Week 2's mini-project.

One thing worth naming upfront: SQL has a purpose-built tool for a lot of this — window functions like `LAG` and moving averages over a defined frame — and you don't have them yet. They're introduced properly in Week 3. Today deliberately does this the harder way, with self-joins, correlated subqueries, and manual date arithmetic, so that when Week 3 hands you the easier tool, you'll actually feel why it's better instead of just being told so.

---

## Step One: Turning Rows Into a Time Series

A time series here just means: one row per time period, with a metric attached. `trips` is one row per ride — the first job is collapsing that into one row per day (or week, or month).

```sql
SELECT
  DATE_TRUNC('day', start_time) AS ride_date,
  COUNT(*) AS total_rides
FROM trips
GROUP BY DATE_TRUNC('day', start_time)
ORDER BY ride_date;
```

This alone is worth looking at closely before doing anything more advanced. Plot it, or scroll through it, and you already have a rough sense of shape — weekly cycles, seasonal swings, and anything that visually jumps out as unusual. Most anomaly detection starts with actually looking, not with a formula.

## Step Two: Detecting Gaps — Missing, Not Just Low

A day with zero rides and a day that's simply *absent from the table* look identical if you only ever `GROUP BY` and count — a missing day just won't produce a row at all, and it's easy to not notice a row that isn't there. You need to compare your actual dates against the *full set of dates that should exist*.

```sql
WITH date_range AS (
  SELECT generate_series(
    (SELECT MIN(DATE_TRUNC('day', start_time)) FROM trips),
    (SELECT MAX(DATE_TRUNC('day', start_time)) FROM trips),
    interval '1 day'
  ) AS calendar_date
),
daily_rides AS (
  SELECT DATE_TRUNC('day', start_time) AS ride_date, COUNT(*) AS total_rides
  FROM trips
  GROUP BY DATE_TRUNC('day', start_time)
)
SELECT dr.calendar_date, COALESCE(d.total_rides, 0) AS total_rides
FROM date_range dr
LEFT JOIN daily_rides d ON dr.calendar_date = d.ride_date
ORDER BY dr.calendar_date;
```

(`generate_series` syntax varies by database — DuckDB and Postgres both support it; check your engine's equivalent if this errors.)

Now every calendar day is represented, and `COALESCE` turns a genuinely missing day into a visible `0` instead of a silent absence. This distinction matters: a `0` from `COALESCE` might mean "no rides happened" or "no data was loaded for this day" — those are very different problems, and only your Week 1–2 data quality notes and judgment can tell them apart. A missing day right at the edge of the 2019/2020 file boundary from Week 1, Day 1 is a strong candidate for a loading gap rather than a real zero.

## Step Three: Period-over-Period Comparison, Without LAG

The natural next question is "how does today compare to yesterday, or this month compare to last month?" Without a window function, you get there with a self-join — joining the time series to a shifted copy of itself.

```sql
SELECT
  today.calendar_date,
  today.total_rides,
  yesterday.total_rides AS previous_day_rides,
  today.total_rides - yesterday.total_rides AS day_over_day_change
FROM daily_series today
JOIN daily_series yesterday
  ON today.calendar_date = yesterday.calendar_date + INTERVAL '1 day';
```

This joins each row to the row exactly one day earlier, giving you both values side by side in the same row. The same pattern extended to a **correlated subquery** gets you a rolling average over a window of days — no self-join needed, but a genuine performance cost since it re-runs per row:

```sql
SELECT
  ds.calendar_date,
  ds.total_rides,
  (
    SELECT AVG(inner_ds.total_rides)
    FROM daily_series inner_ds
    WHERE inner_ds.calendar_date BETWEEN ds.calendar_date - INTERVAL '6 days' AND ds.calendar_date
  ) AS trailing_7day_avg
FROM daily_series ds
ORDER BY ds.calendar_date;
```

This is the manual version of a 7-day moving average. It works, but notice what it costs: for every single row, the database re-scans a 7-day window from scratch. This is exactly the kind of thing Week 3's window functions replace with something both shorter to write and faster to run.

## Step Four: Flagging Statistical Anomalies

Once you have a metric per period, "anomalous" needs a working definition — not just "looks weird." A standard, defensible approach: compute the mean and standard deviation of the metric, then flag periods that fall far enough from the mean to be worth a second look.

```sql
WITH stats AS (
  SELECT AVG(total_rides) AS mean_rides, STDDEV(total_rides) AS stddev_rides
  FROM daily_series
)
SELECT
  ds.calendar_date,
  ds.total_rides,
  (ds.total_rides - stats.mean_rides) / NULLIF(stats.stddev_rides, 0) AS z_score,
  CASE
    WHEN ABS((ds.total_rides - stats.mean_rides) / NULLIF(stats.stddev_rides, 0)) > 3 THEN 'Extreme'
    WHEN ABS((ds.total_rides - stats.mean_rides) / NULLIF(stats.stddev_rides, 0)) > 2 THEN 'Notable'
    ELSE 'Normal'
  END AS anomaly_flag
FROM daily_series ds
CROSS JOIN stats
ORDER BY ds.calendar_date;
```

The **z-score** here says how many standard deviations a day's ride count is from the overall average. `NULLIF(stats.stddev_rides, 0)` guards against a division error if, in some edge case, the standard deviation were exactly zero. A threshold of 2–3 standard deviations is a common starting convention, not a law — tighten or loosen it depending on how many flagged periods you actually want to review.

One real limitation worth naming: computing the mean and standard deviation across *all seven years at once* treats a normal Tuesday in January the same as a normal Saturday in July, when ridership genuinely differs by season and day of week. A more careful version would compute separate statistics per month or per day-of-week before flagging — something worth trying in today's task file, and something window functions in Week 3 make considerably easier to do well.

## Distinguishing a Genuine Trend From a Gap or a One-Off

A flagged anomaly isn't an answer — it's a starting point for a judgment call, and that judgment call has a rough process:

1. **Check for a data gap first.** Does the anomalous period line up with a missing-date gap from Step Two, a schema-era boundary from Week 1, Day 1, or the unmatched-station issue from Day 3? If so, it may not be a real signal at all.
2. **Check whether it persists.** A single anomalous day that reverts immediately afterward is more likely a one-off event (weather, a holiday, a service disruption) than a genuine shift. A change that holds across weeks or months is more likely a real trend.
3. **Check for a corroborating signal.** This dataset includes NOAA weather data alongside the trip records — a ridership drop that lines up with a major weather event has an explanation outside the data itself, which is different from a drop with no external cause.
4. **Check whether it's a known real-world event.** This dataset spans 2019–2026, which includes some genuinely disruptive real-world periods for public transit and shared mobility generally. A sustained, unexplained shift in a specific window is worth checking against what was actually happening at that time, rather than assumed to be a data artifact by default.

None of this fully automates. The SQL gets you to a short list of periods worth a closer look; deciding what each one actually means is the analytical work the mini-project asks for.

---

## What's Next

Today's task file builds the full pipeline described above against `trips`, then asks you to actually investigate two or three flagged periods and write up what you find — the Week 2 mini-project. From there, Week 3 introduces window functions, which replace most of today's self-joins and correlated subqueries with something shorter and considerably faster, and goes further into the kind of professional-level SQL that a real data team would use for exactly this problem.