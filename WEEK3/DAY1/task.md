# Day 1 Task: Window Function Fundamentals

**Total time: 6 hours**

Week 2 covered a working toolkit for comparing rows to each other: self-joins, correlated subqueries, chained CTEs. Today replaces those with window functions in the cases where a window function does the same job with less code. You'll rebuild several Week 1 and Week 2 answers using `OVER()`, `PARTITION BY`, `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LAG()`, and `LEAD()` — and directly compare the result to what you wrote before.

Read `week3_day1_concepts.md` before starting if you haven't already — this task file assumes you understand the difference between `ROW_NUMBER`, `RANK`, and `DENSE_RANK`, and what `PARTITION BY` does to a window function's scope.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | OVER() basics and ROW_NUMBER | 60 min |
| 2 | RANK and DENSE_RANK | 60 min |
| 3 | PARTITION BY | 75 min |
| 4 | LAG and LEAD | 75 min |
| 5 | Rewriting Week 2's self-joins | 60 min |
| 6 | Reflection | 30 min |

---

## Block 1: OVER() Basics and ROW_NUMBER (60 min)

**Goal:** Get comfortable with the empty `OVER()` case and basic row numbering before adding any complexity.

### Do this: 6 tasks

1. Write a query attaching the overall average station capacity to every row using `AVG(capacity) OVER ()`, and compare it to your Week 1 scalar-subquery version of the same thing.
2. Add `ROW_NUMBER() OVER (ORDER BY capacity DESC)` to the same query, numbering every station from highest to lowest capacity.
3. Confirm that removing the `ORDER BY` inside `OVER()` still runs without erroring, but produces row numbers in a database-determined, unreliable order. Note what you observe.
4. Use `ROW_NUMBER()` to number stations by capacity ascending instead of descending, and confirm the numbers reverse as expected.
5. Combine two window functions in one query: `ROW_NUMBER()` for rank and `AVG(capacity) OVER ()` for the overall average, both alongside the raw station data.
6. Write one sentence comparing how this felt to write versus the equivalent Week 1 subquery version — be specific about what got shorter or clearer.

---

## Block 2: RANK and DENSE_RANK (60 min)

**Goal:** See ties actually happen and understand why the two functions diverge afterward.

### Do this: 7 tasks

1. Find (or construct, if none exist naturally) a set of stations that tie on capacity, and run `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` side by side over the same `ORDER BY`.
2. Confirm directly, from the output, that `ROW_NUMBER()` breaks the tie arbitrarily while `RANK()` and `DENSE_RANK()` both assign the same value to tied rows.
3. Confirm directly that `RANK()` skips a number after a tie while `DENSE_RANK()` doesn't — find the specific row where the two diverge and explain why in a sentence.
4. Apply `RANK()` to `stations_summary`, ranking stations by `total_rides` descending, and pull just the top 5 ranks.
5. Decide which of the three functions is actually correct for a "top 5 busiest stations" report if there's a tie for 5th place — would you want 5 rows, 6, or something else — and pick the function that produces that behavior.
6. Try using `RANK()` or `DENSE_RANK()` results directly in a `WHERE` clause (for example, `WHERE rank_num <= 5`) and confirm you get an error — window functions can't be filtered directly in the same query level they're computed in.
7. Fix the Task 6 error by wrapping the window function query in a subquery or CTE and filtering in the outer query instead. Note that this is the same pattern Day 1, Week 3's concepts file used for "top N per group."

---

## Block 3: PARTITION BY (75 min)

**Goal:** Scope window functions per group without collapsing any rows.

### Do this: 7 tasks

1. Rewrite the neighborhood-average-capacity query from the concepts file yourself, using `AVG(capacity) OVER (PARTITION BY neighborhood)`.
2. Add a column computing the difference between each station's capacity and its neighborhood's average, using the `PARTITION BY` result directly in the same `SELECT`.
3. Rebuild Week 1, Day 4, Block 4's "stations above their neighborhood average" — this time using `PARTITION BY` and a `WHERE` clause on a wrapped subquery, instead of the correlated subquery version from that day.
4. Rebuild Week 2, Day 2, Block 4's mini-project ("busiest station per neighborhood") using `ROW_NUMBER() OVER (PARTITION BY neighborhood ORDER BY total_rides DESC)` and a filter for `rn = 1`.
5. Compare your window-function version of Task 4 against the three-stage chained-CTE version you built in Week 2. Which one would you rather maintain?
6. Try partitioning by two columns at once (for example, `PARTITION BY neighborhood, rider_type` if that combination makes sense against your joined data), and confirm each unique combination gets its own window.
7. Write a query with two different `PARTITION BY` window functions in the same `SELECT` — one partitioned by neighborhood, one partitioned by rider type — and confirm both work independently in the same result set.

---

## Block 4: LAG and LEAD (75 min)

**Goal:** Compare each row to its neighbors without a self-join.

### Do this: 8 tasks

1. Rebuild the daily time series from Week 2, Day 4, and add `LAG(total_rides) OVER (ORDER BY calendar_date)` to pull each day's previous value.
2. Compute the day-over-day change as `total_rides - LAG(total_rides) OVER (ORDER BY calendar_date)`, and compare the result directly against your Week 2, Day 4 self-join version.
3. Use `LEAD()` instead of `LAG()` to compute the *next* day's ride count alongside each row, and confirm the values shift in the opposite direction from `LAG()`.
4. Use `LAG(total_rides, 7)` to compare each day to the same day one week earlier, replacing the modified self-join from Week 2, Day 4, Block 3, Task 4.
5. Add a default value as the third argument to one of your `LAG()` calls (for example, `LAG(total_rides, 1, 0)`), and confirm the very first row now shows `0` instead of `NULL`.
6. Use `LAG()` combined with `PARTITION BY` — for example, comparing each station's ride count to its own previous period, partitioned by station, if you build a per-station daily series. Confirm the partition boundary correctly resets what counts as "previous."
7. Find the largest single day-over-day increase and decrease using your `LAG()`-based query, and confirm the results match what you found in Week 2, Day 4 using the self-join.
8. Write a sentence comparing how confident you feel in the `LAG()` version versus the self-join version — which one would you trust more if you found it in someone else's code six months from now, and why?

---

## Block 5: Rewriting Week 2's Self-Joins (60 min)

**Goal:** Directly replace the heaviest patterns from Week 2, Day 4 with window functions, end to end.

### Do this: 5 tasks

1. Take your full Week 2, Day 4 anomaly-detection pipeline — daily series, gap detection, period comparison, and statistical flagging — and rebuild the period-comparison portion entirely with `LAG()`.
2. Rebuild the trailing 7-day moving average from Week 2, Day 4's correlated subquery using a window function instead: `AVG(total_rides) OVER (ORDER BY calendar_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)`. (This frame syntax is a preview of Day 2 — try it now and don't worry if it's not fully clear yet.)
3. Compare the moving-average results from Task 2 against your original Week 2, Day 4 correlated-subquery version, row for row, to confirm they match.
4. Note, specifically, whether the window-function version ran noticeably faster than the correlated-subquery version on the full seven-year dataset, if your tool shows query timing.
5. Leave the statistical z-score flagging from Week 2, Day 4 as-is for now — it doesn't need a window function, and Day 2 will revisit whether any part of it could still benefit from one.

---

## Block 6: Reflection (30 min)

Answer in a few sentences each:

1. Pick one query you rebuilt today and had also built in Week 1 or Week 2 using a subquery, join, or self-join. Set both versions side by side. Beyond just being shorter, what changed about how easy the query is to actually trust at a glance?
2. Block 2 showed `RANK()` and `DENSE_RANK()` diverging on ties. Pick a real question from this dataset where the choice between them would actually change your answer, not just the row numbers.
3. Block 4, Task 8 asked which version you'd trust more in someone else's code. Now flip it: if you were the one leaving code for someone else, would you always default to the window function version, or is there a case from today where the more explicit self-join or subquery might still communicate the intent better?
4. The Day 1 concepts file closed with a note that today's `SELECT` statements are close in shape to what a dbt model looks like. Pick one query you wrote today and, in a sentence, describe what you think would need to change (if anything) for it to live on its own as a single `.sql` file that other queries could build on top of.
5. Tomorrow extends today's `OVER()` clause with window **frames** — controlling exactly which nearby rows a function considers, beyond just partitioning — and adds a new weather dataset to measure volatility, not just averages. Based on Block 5, Task 2's preview, what do you expect `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` is actually doing, in your own words?

**Deliverable:** Your rebuilt version of Week 2, Day 2's "busiest station per neighborhood" using `ROW_NUMBER()` and `PARTITION BY`, and your rebuilt version of Week 2, Day 4's period-over-period comparison using `LAG()` — each with a one-line comment noting which earlier approach it replaces.