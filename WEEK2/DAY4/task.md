# Day 4: Detecting Anomalies & the Week 2 Mini-Project

**Total time: 6 hours**

This is the last day of Week 2, and it closes with the mini-project described back at the top of this week: identify a meaningful trend in the data, and separate genuine change from data gaps or one-off anomalies. Everything today builds toward that — turning `trips` into a real time series, finding the gaps in it, comparing periods, flagging statistical outliers, and then actually investigating a couple of them instead of just listing them.

Read `day4_concepts.md` before starting if you haven't already — this task file assumes you understand `generate_series`, self-joins for period-over-period comparison, and z-score-based flagging.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Building a daily time series | 60 min |
| 2 | Detecting missing periods | 75 min |
| 3 | Period-over-period comparison | 75 min |
| 4 | Flagging statistical anomalies | 75 min |
| 5 | Mini-project: investigate and write up | 45 min |
| 6 | Reflection | 30 min |

---

## Block 1: Building a Daily Time Series (60 min)

**Goal:** Collapse `trips` into one row per day, and actually look at it before doing anything else.

### Do this: 6 tasks

1. Build the daily ride-count time series from the concepts file, grouping `trips` by `DATE_TRUNC('day', start_time)`.
2. Order it chronologically and scroll through at least a full year of it. Note anything that visually jumps out — a sharp drop, a sudden spike, an unusually flat stretch.
3. Build the same time series at a weekly grain instead of daily, using your database's week-truncation function. Compare how much noisier the daily version looks next to the smoother weekly one.
4. Compute the overall minimum, maximum, and average daily ride count across all seven years.
5. Filter the daily series to just 2020 and scan it specifically — this year is worth a closer look later in this task, given what was happening globally that year.
6. Write one or two sentences on what you noticed just from looking, before running any statistical flagging — first impressions are worth recording before they get replaced by the formal analysis.

---

## Block 2: Detecting Missing Periods (75 min)

**Goal:** Separate "zero rides" from "no data at all."

### Do this: 7 tasks

1. Build the full calendar-date range for your dataset using `generate_series` (or your engine's equivalent), from the earliest to the latest date in `trips`.
2. `LEFT JOIN` your daily ride-count series onto that full calendar, and use `COALESCE` to turn genuinely missing days into a visible `0`.
3. Filter down to just the days where the original (pre-`COALESCE`) ride count was `NULL` — these are true data gaps, not real zero-ride days.
4. Cross-reference those gap dates against your Week 1, Day 1 notes on schema eras and file boundaries. Do any gaps line up with where one year's file structure changed to another's?
5. Cross-reference the gap dates against your Day 3 notes on the unmatched-station issue — is there any relationship, or are these separate problems?
6. Count how many total gap days you found, and what percentage of the full date range that represents.
7. Decide, and write a sentence justifying it: should gap days be excluded from later analysis entirely, or included as `0`s? Note that this decision will directly affect Block 4's anomaly flagging if left as `0`.

---

## Block 3: Period-over-Period Comparison (75 min)

**Goal:** Compare each period to the one before it, without a window function.

### Do this: 6 tasks

1. Build the self-join from the concepts file, comparing each day's ride count to the previous day's.
2. Find the single largest day-over-day increase and the single largest day-over-day decrease across the full dataset.
3. Investigate both of those extreme days directly — what day of week were they, what time of year, and does anything in your gap list from Block 2 explain either one?
4. Extend the self-join to compare each day to the same day one week earlier, instead of one day earlier — this controls for the weekly cycle noted in Block 1.
5. Build the correlated-subquery trailing 7-day average from the concepts file, and compare a handful of individual days against their trailing average rather than the single previous day.
6. Note, from direct experience now, one specific way the trailing-average correlated subquery felt more cumbersome than the self-join comparison — this is useful context for appreciating window functions in Week 3.

---

## Block 4: Flagging Statistical Anomalies (75 min)

**Goal:** Turn "looks unusual" into a defensible, threshold-based flag.

### Do this: 7 tasks

1. Compute the overall mean and standard deviation of daily ride counts, and build the z-score and `CASE WHEN` flag from the concepts file.
2. List every day flagged as `'Extreme'` (beyond 3 standard deviations), sorted by date.
3. List every day flagged as `'Notable'` (beyond 2 but within 3 standard deviations), and note roughly how many there are compared to `'Extreme'`.
4. Revisit the concepts file's limitation about computing one global mean and standard deviation across all seasons and days of week. Rebuild the stats CTE to compute the mean and standard deviation *per month* instead, and reflag anomalies against their own month's statistics.
5. Compare your flagged-day list before and after Task 4's per-month adjustment. Did any days drop off the list, or get added, once seasonality was accounted for?
6. Cross-reference your final flagged-day list against the gap-day list from Block 2. Remove or separately label any flagged day that's actually a data gap rather than a real anomaly.
7. Narrow your remaining flagged list down to two or three days or periods that seem like genuine, worth-investigating anomalies — these carry into Block 5.

---

## Block 5: Mini-Project — Investigate and Write Up (45 min)

**Goal:** Turn your flagged periods into an actual finding, following the four-step judgment process from the concepts file.

### Do this: 6 tasks

1. For each of your two or three flagged periods from Block 4, check whether it lines up with a data gap or schema boundary you've already documented.
2. For each remaining period, check whether the change persisted over multiple days or weeks, or reverted immediately — pull the surrounding week of data to see.
3. If your database includes the joined NOAA weather data mentioned in the course description, check whether any flagged period lines up with a significant weather event. If you don't have weather data loaded, note that as a limitation of this analysis rather than skipping the question.
4. For any period that still looks like a genuine, real-world event rather than a data or weather artifact, research briefly what was happening in that specific window that could plausibly explain a shift in bikeshare ridership.
5. Write a short report (half a page to a page) identifying one meaningful trend or anomaly in the data, and explicitly walking through how you ruled out — or ruled in — a data gap, a weather effect, and a known external event as the explanation.
6. Save this write-up as `notes/week2_trend_analysis.md` and commit it to your GitHub repo from Week 1, Day 5, alongside the SQL that produced the underlying flagged-period list.

---

## Block 6: Reflection (30 min)

Answer in a few sentences each:

1. Block 2 asked you to decide whether gap days should be excluded or treated as `0`. Now that you've run the full anomaly-flagging pipeline, did that early decision end up mattering for your final results — did any gap day almost get mistaken for a real anomaly?
2. Block 4 showed your flagged-day list changing once you moved from one global standard deviation to per-month standard deviations. What does that tell you about the risk of applying a single statistical threshold to a dataset with strong seasonal patterns?
3. Look back at Week 2 as a whole — subqueries, CTEs, NULL handling, and now anomaly detection. Which of those four tools did you end up leaning on most heavily today, and did you expect that going in?
4. This week's tools got you a defensible flagged list, but Block 5 still required real research and judgment to turn a flagged date into an actual explained finding. What does that tell you about where SQL's job ends and an analyst's job begins?
5. Week 3 introduces window functions, which the concepts file said would replace most of today's self-joins and correlated subqueries. Based on today's experience, which specific task from this file are you most looking forward to redoing with a window function instead?

**Deliverable:** The full anomaly-detection SQL (time series, gap detection, and per-month statistical flagging), plus `notes/week2_trend_analysis.md`, committed to your GitHub repo — this is the Week 2 mini-project deliverable in full.