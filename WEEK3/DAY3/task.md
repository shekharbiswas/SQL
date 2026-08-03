# Day 3 Task: Recursive CTEs, Query Performance & Pivoting

**Total time: 6 hours**

Today covers three separate topics rather than one continuous thread: building a recursive CTE from scratch, reading `EXPLAIN` output on real queries from this course, and reshaping data both directions with pivot and unpivot. None of today builds directly on Days 1–2's window functions, though the reflection at the end ties all three days together before Day 4 changes the subject entirely.

Read `week3_day3_concepts.md` before starting if you haven't already — this task file assumes you understand the anchor/recursive-member structure of `WITH RECURSIVE`, the difference between a sequential scan and an index scan, and the basic shape of `PIVOT`/`UNPIVOT`.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Recursive CTEs: building a date spine | 90 min |
| 2 | Recursive CTEs: bike movement chains | 60 min |
| 3 | Reading EXPLAIN plans and indexing | 75 min |
| 4 | Pivoting | 60 min |
| 5 | Unpivoting | 45 min |
| 6 | Reflection | 30 min |

---

## Block 1: Recursive CTEs — Building a Date Spine (90 min)

**Goal:** Understand exactly how a recursive CTE builds up a result set, iteration by iteration, using the table you've already been querying all week.

### Do this: 7 tasks

1. Write the date-spine recursive CTE from the concepts file yourself, generating one row per date across the full range your `daily_series` table covers.
2. Confirm your generated spine has exactly the right number of rows for the date range (a quick way to check: the number of days between your start and end date, inclusive).
3. Deliberately remove the `WHERE` condition from the recursive member and, in a **separate** query with an explicit `LIMIT` added as a safety valve, observe what happens. Note how many rows you get back and why the recursion didn't stop on its own.
4. Left join your generated date spine against the real `daily_series` table on `calendar_date`, and confirm this reproduces the same gap-detection result you built in Week 2, Day 2 — any spine date with no matching real row is a gap.
5. Modify the recursive CTE to step by week instead of by day (`calendar_date + INTERVAL 7 DAY`), and confirm the resulting spine has roughly one-seventh as many rows.
6. Build a second recursive CTE that generates a spine of month-start dates instead of individual days, useful for month-over-month reporting without a full daily spine.
7. Write a sentence explaining, in your own words, why a recursive CTE was necessary here instead of some non-recursive alternative — what specifically can't a plain `SELECT`, `GROUP BY`, or window function do that this needed?

---

## Block 2: Recursive CTEs — Bike Movement Chains (60 min)

**Goal:** Use a recursive CTE for a genuinely hierarchical relationship, where one row's output determines the next row's input.

### Do this: 6 tasks

1. Pick a specific `bike_id` and date with several trips recorded, and build the bike-chain recursive CTE from the concepts file to reconstruct that bike's full path across the day.
2. Confirm your `chain_position` column correctly numbers each trip in sequence, starting from 1.
3. Confirm the chain naturally stops when there's no next trip whose `start_station` matches the current link's `end_station` — note what that means for a bike that was, for example, picked up and moved by rebalancing staff rather than another rider.
4. Find a bike with a broken chain — a trip whose `end_station` doesn't match the next trip's `start_station` for that bike — and note how many separate chains that bike actually produces on that day as a result.
5. Modify your recursive CTE to also track the total elapsed time from the first trip in the chain to the current link's trip, using `start_time` from the anchor member carried through each recursive step.
6. Write a sentence on whether you think this kind of chain-reconstruction is something you'd expect to need often in real bikeshare analysis, or whether it's a niche case — and what business question would actually justify building it.

---

## Block 3: Reading EXPLAIN Plans and Indexing (75 min)

**Goal:** Get comfortable reading a query plan and reasoning about why one version of a query might be faster than another.

### Do this: 7 tasks

1. Run `EXPLAIN` on a simple filtered query against `trips` (for example, `WHERE start_station = '...'`), and identify whether the plan uses a sequential scan or an index scan.
2. If your tool supports `EXPLAIN ANALYZE`, run it on the same query and compare the estimated row count against the actual row count. Note whether they're close or far apart.
3. Run `EXPLAIN` on one of your Day 1 or Day 2 window function queries (for example, the trailing 7-day moving average), and identify any step in the plan related to sorting — window functions with an `ORDER BY` inside `OVER()` typically require the data to be sorted first.
4. Compare `EXPLAIN` output for your Week 2, Day 4 correlated-subquery trailing average against `EXPLAIN` output for the Day 2 window-function version of the same query. Note any visible difference in the plan's structure, not just a runtime number.
5. If your tool exposes actual timing (`EXPLAIN ANALYZE` or similar), find the single slowest query you've written so far this course and identify, from the plan, which specific step accounts for most of the time.
6. Read a short note or your engine's documentation on how indexing concepts apply specifically to the tool you're using this course (DuckDB via Beekeeper Studio), and write two or three sentences on what, if anything, changes about how you'd think about "should I add an index" in a columnar engine versus a traditional row-store database.
7. Pick one query from this week you suspect could be written a genuinely faster way, and use `EXPLAIN` (or `EXPLAIN ANALYZE`) to check whether an alternative phrasing actually produces a different — hopefully cheaper — plan.

---

## Block 4: Pivoting (60 min)

**Goal:** Turn long trip data into a wide report-ready shape, both with native syntax and by hand.

### Do this: 6 tasks

1. Build a monthly trip-count-by-rider-type pivot using DuckDB's native `PIVOT` syntax, producing one row per month with a separate column per rider type.
2. Rebuild the same result using conditional aggregation (`SUM(CASE WHEN ... THEN 1 ELSE 0 END)`), and confirm the two versions produce identical numbers.
3. Extend your pivot to a second dimension you haven't pivoted on yet this course — for example, trip counts by weekday, pivoted so each weekday is its own column.
4. Pivot the weather data instead of the trip data: build one row per weather station with a column for each month's average `temp_high`.
5. Try pivoting on a column with a large number of distinct values (for example, `station_name`, if your dataset has dozens of stations) and note what happens to the readability of the result — at what point does a pivot stop being useful for a human reader?
6. Write a sentence on which of today's two pivot approaches (native `PIVOT` vs. conditional aggregation) you'd actually reach for in your own work, and why.

---

## Block 5: Unpivoting (45 min)

**Goal:** Reshape wide data back into long format, and recognize when that's actually the necessary first step rather than the last.

### Do this: 5 tasks

1. Take the pivoted monthly rider-type table you built in Block 4 and unpivot it back into long format using DuckDB's `UNPIVOT` syntax, confirming you recover the original row-per-`(month, rider_type)` shape.
2. Confirm the round trip is lossless: aggregate your original long-format data and your pivoted-then-unpivoted data the same way, and confirm the totals match exactly.
3. Imagine (or, if you have one, use) a hypothetical wide weather export with one column per month (`jan_avg_temp`, `feb_avg_temp`, ...) instead of a `weather_date` row per observation, and write the `UNPIVOT` query that would turn it into a proper long format with one row per month.
4. Explain, in a sentence, why that long format from Task 3 would need to exist *before* you could apply any of this week's window functions to it — what specifically about `OVER (ORDER BY ...)` requires one row per time period rather than one column per time period?
5. Write a sentence on a real or plausible source file you might actually receive in a work context — a spreadsheet export, a vendor report — that would arrive wide and need unpivoting before anything else in this course's toolkit could touch it.

---

## Block 6: Reflection (30 min)

Answer in a few sentences each:

1. Of today's three topics — recursive CTEs, `EXPLAIN`/indexing, pivot/unpivot — which one do you expect to use most often in real work, and which one feels most like a tool you'll reach for rarely but need to recognize when you see it in someone else's code?
2. Block 1 showed a recursive CTE without a terminating condition running away. Compare that failure mode to a mistake you might make with a regular `JOIN` or `GROUP BY` — is a broken recursive CTE more dangerous, less dangerous, or just differently dangerous than an ordinary SQL bug?
3. Block 3 asked you to compare `EXPLAIN` output between a correlated subquery and its window-function replacement. Setting timing aside, did *reading the plan itself* change how you think about why window functions have been faster this week, or was the improvement more about what you already understood from Days 1–2?
4. The concepts file suggested the date-spine recursive CTE from Block 1 is a strong candidate for its own dbt model, referenced by name instead of rebuilt in every query that needs a gap-free calendar. Now that you've built it by hand, does that suggestion make more or less sense to you than it did before you wrote the query yourself?
5. Tomorrow introduces dbt directly, using this week's SQL as the starting material. Based on everything you've built across Week 1 through today — subqueries, joins, CTEs, window functions, a recursive date spine — what's one specific piece of repeated logic from this course that you'd most want turned into something reusable, if you had to guess before seeing how dbt actually does it?

**Deliverable:** Your working date-spine recursive CTE joined against `daily_series` to reproduce Week 2, Day 2's gap detection; your bike-chain recursive CTE for one specific bike and date; and your monthly rider-type report built both ways — once with native `PIVOT`, once with conditional aggregation — committed to your GitHub repo with a one-line comment on each file noting what it replaces or demonstrates.