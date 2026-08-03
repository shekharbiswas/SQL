# Day 2 Task: Running Totals, Moving Averages, Percent Change & Weather Volatility

**Total time: 6 hours**

Yesterday replaced Week 2's self-joins with `LAG()` and `LEAD()`. Today replaces the correlated subquery that computed a trailing 7-day average back in Week 2, Day 4, builds running totals and percent-change comparisons using frame clauses, and brings in a new weather dataset to measure volatility, not just averages, at the event-location level. By the end of today, your Week 2 anomaly-detection pipeline should be running entirely on window functions, with no self-joins or correlated subqueries left in it.

Read `week3_day2_concepts.md` before starting if you haven't already — this task file assumes you understand frame boundaries (`PRECEDING`, `CURRENT ROW`, `FOLLOWING`), the default frame trap, the difference between `ROWS` and `RANGE`, and how `STDDEV()` behaves as a window function.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Frame syntax and running totals | 60 min |
| 2 | Trailing moving averages | 60 min |
| 3 | Centered averages and ROWS vs RANGE | 45 min |
| 4 | Period-over-period percent change | 60 min |
| 5 | Weather volatility: STDDEV and the trip-weather join | 60 min |
| 6 | Rebuilding the Week 2 anomaly pipeline | 45 min |
| 7 | Reflection | 30 min |

---

## Block 1: Frame Syntax and Running Totals (60 min)

**Goal:** Get the default-frame behavior and explicit `ROWS BETWEEN` syntax both under control.

### Do this: 6 tasks

1. Build a running total of daily rides using `SUM(total_rides) OVER (ORDER BY calendar_date)` with no explicit frame, and confirm the last row's value equals the total ride count across the whole dataset.
2. Rewrite Task 1 with an explicit `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` frame, and confirm the results are identical to the default-frame version.
3. Try `AVG(total_rides) OVER (ORDER BY calendar_date)` with no explicit frame, and confirm — by comparing the first and last rows — that you got a *running* average, not an overall one. This is the default-frame trap from the concepts file, now demonstrated directly.
4. Fix Task 3 by adding an explicit `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` frame, and confirm every row now shows the same, correct overall average.
5. Build a running total of rides per station using `PARTITION BY station_name` alongside `ORDER BY calendar_date`, confirming the total resets at each new station.
6. Pick a specific date partway through the dataset and manually verify its running total by summing the raw daily values up through that date some other way (a simple filtered `SUM`), confirming your window function's result is correct.

---

## Block 2: Trailing Moving Averages (60 min)

**Goal:** Replace Week 2, Day 4's correlated subquery for real, and confirm it produces identical results.

### Do this: 6 tasks

1. Build a trailing 7-day moving average using `AVG(total_rides) OVER (ORDER BY calendar_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)`.
2. Compare this row-by-row against your Week 2, Day 4 correlated-subquery trailing average. Confirm they match exactly.
3. Build a trailing 30-day moving average using the same pattern with a different `PRECEDING` value, and compare its smoothness against the 7-day version by scanning both visually.
4. Check what happens at the very start of the dataset, where fewer than 7 (or 30) prior rows exist. Confirm the frame quietly uses however many rows are actually available rather than erroring or returning `NULL`.
5. Try building the same trailing average with `PARTITION BY station_name` added, producing a per-station trailing average instead of a dataset-wide one.
6. Note, specifically, any difference in how long this window-function version takes to run compared to the Week 2 correlated-subquery version, if your tool exposes timing.

---

## Block 3: Centered Averages and ROWS vs. RANGE (45 min)

**Goal:** Build a centered window and see the ROWS/RANGE distinction on data that actually has ties.

### Do this: 5 tasks

1. Build a centered 7-day average using `ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING`, and compare its shape against the trailing 7-day version from Block 2 — the centered version should look smoother and less "delayed" relative to real spikes.
2. Confirm that the very first and last few rows of the centered average have a smaller effective window, since there aren't 3 full rows available on one side.
3. Build a query with a column that genuinely has duplicate values at the row level — for example, trips truncated to the hour, where multiple trips can share the same hour value — and rank them using `ROWS BETWEEN` vs an equivalent `RANGE BETWEEN` frame.
4. Confirm directly, from the output, where `ROWS` and `RANGE` diverge on that tied data — pick one specific row and explain the different result each produces.
5. Decide, for the trip-duration or ride-count reporting you've built this week, whether `ROWS` or `RANGE` is the correct default going forward, and write a sentence justifying it.

---

## Block 4: Period-over-Period Percent Change (60 min)

**Goal:** Turn raw comparisons into percentages, and handle the zero-division edge case correctly.

### Do this: 6 tasks

1. Build the day-over-day percent change query from the concepts file, using `LAG()` and `NULLIF` to guard against division by zero.
2. Find the single largest positive and negative percent change in the dataset, and check whether either lines up with a flagged anomaly from Week 2, Day 4.
3. Build a week-over-week percent change instead, using `LAG(total_rides, 7)` as the comparison point.
4. Build a month-over-month percent change using a monthly-aggregated version of your time series (grouping by `DATE_TRUNC('month', ...)` first) rather than the daily series.
5. Confirm your `NULLIF` guard actually works by checking whether any day in your dataset has a previous value of exactly zero, and confirming that row shows `NULL` for percent change instead of erroring.
6. Round your percent-change values to one decimal place using `ROUND()`, and confirm the output is genuinely more readable than the raw unrounded version.

---

## Block 5: Weather Volatility — STDDEV and the Trip-Weather Join (60 min)

**Goal:** Join trips to weather at the event-location level, then measure volatility with `STDDEV()` the same way earlier blocks measured center with `AVG()`.

### Do this: 7 tasks

1. Build the three-table join from the concepts file — `station_daily_series`, `stations`, and `daily_weather` — matching each station's daily rides to its `nearest_weather_station_id`'s observations for the same date. Confirm each row now carries both ride counts and weather values for the correct location.
2. Add `STDDEV(temp_high) OVER (PARTITION BY station_name ORDER BY calendar_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)` to compute a trailing 7-day temperature volatility per station.
3. Add a matching `STDDEV(daily_rides) OVER (...)` using the same partition and frame, so each row shows both trailing weather volatility and trailing ridership volatility side by side.
4. Pick one station and scan for a week where temperature volatility was unusually high. Check whether ridership volatility was also elevated that same week, or whether the two moved independently.
5. Recompute one of your Block 5 `STDDEV()` columns using `STDDEV_POP()` instead of the default `STDDEV_SAMP()`, and confirm the two values are close but not identical — note by how much they diverge on a window with only a handful of rows versus a full 30-day window.
6. Try `AVG(temp_high) OVER (...)` with no explicit frame in the same query as a `STDDEV()` window function that does have an explicit frame, and confirm the default-frame trap from Block 1 applies here too if you're not careful.
7. Write two or three sentences: does trailing weather volatility look like it explains any of the ridership anomalies flagged in Week 2, Day 4, or do those anomalies look driven by something else? You don't need a rigorous answer — a specific, evidence-based guess is the goal.

---

## Block 6: Rebuilding the Week 2 Anomaly Pipeline (45 min)

**Goal:** Fully retire the self-join and correlated subquery from Week 2, Day 4.

### Do this: 5 tasks

1. Open your Week 2, Day 4 anomaly-detection SQL file and identify every self-join and correlated subquery still in it.
2. Replace the day-over-day self-join with `LAG()` (from Day 1) if you haven't already.
3. Replace the trailing 7-day correlated subquery with the `AVG() OVER (... ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)` version from Block 2.
4. Re-run the full pipeline end to end and confirm your flagged-anomaly list from Week 2, Day 4 is unchanged — the results should be identical, only the method should differ.
5. Update `sql/day4_anomaly_detection.sql` (or wherever this lives in your Week 2 folder structure) with the window-function version, and commit it with a message noting it replaces the self-join/correlated-subquery approach.

---

## Block 7: Reflection (30 min)

Answer in a few sentences each:

1. Block 1, Task 3 showed `AVG() OVER (ORDER BY ...)` silently defaulting to a running average instead of an overall one. Now that you've seen it happen on real data, how would you catch this kind of mistake in your own work going forward, before it reached a report?
2. Compare the readability of Block 4's percent-change query against something you might have tried to write in Week 1 or Week 2 for the same question, before you had `LAG()` available at all. Is there a version of "percent change from the previous period" that's even reasonably expressible without a window function?
3. Block 5 asked whether weather volatility explains any ridership anomalies. What would you need — another column, another dataset, a different statistical test — to actually answer that question with more confidence than a scan-and-guess?
4. Block 6 asked you to confirm the rebuilt anomaly pipeline produces identical results to the Week 2 original. Why does that check matter here specifically — what would it mean if the window-function version had quietly produced different numbers?
5. The concepts file suggested the trip-weather join from Block 5 is a natural candidate for its own dbt model, so it isn't rewritten in every query that needs it. Setting dbt itself aside, if you had to reuse that exact three-table join five more times this week using only what you know today (CTEs, views), which would you reach for, and why?
6. Tomorrow moves into a genuinely different set of tools — recursive CTEs, reading `EXPLAIN` plans, and pivoting/unpivoting — none of which are about comparing rows to nearby rows the way this week has been so far. Based on the bikeshare and weather data as you understand it now, can you think of a question in this data that might actually need a recursive structure?

**Deliverable:** The updated `sql/day4_anomaly_detection.sql` from Week 2, now running entirely on window functions with no self-joins or correlated subqueries, committed to your GitHub repo — plus your Block 5 trip-weather volatility query (trailing `STDDEV()` of both temperature and ridership, partitioned by station) and your Block 4 month-over-month percent-change query.