# Day 5 Task: Tests, Documentation, Macros & Shipping a Scheduled Pipeline

**Total time: 6 hours**

Yesterday's dbt project runs correctly but doesn't check itself, isn't documented, repeats some logic across models, rebuilds its largest table from scratch every run, and only runs when you type a command. Today fixes all five, in that order, and ends with a GitHub Actions workflow that runs the whole project on a schedule without you.

Read `week3_day5_concepts.md` before starting if you haven't already — this task file assumes you understand generic vs. singular tests, what `dbt docs generate` produces, how a macro is defined and called, and what `is_incremental()` and `{{ this }}` do inside an incremental model.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Generic tests | 60 min |
| 2 | A singular custom test | 45 min |
| 3 | Documentation | 60 min |
| 4 | A reusable macro | 60 min |
| 5 | An incremental model | 60 min |
| 6 | Scheduling the pipeline | 45 min |
| 7 | Reflection | 30 min |

---

## Block 1: Generic Tests (60 min)

**Goal:** Add `not_null`, `unique`, `accepted_values`, and `relationships` tests to yesterday's staging and mart models, and see one actually fail.

### Do this: 7 tasks

1. Add `unique` and `not_null` tests on `trip_id` in `stg_trips`'s `schema.yml`.
2. Add an `accepted_values` test on `rider_type` in `stg_trips`, listing every rider type value that actually appears in your data.
3. Add a `relationships` test on `stg_trips.start_station`, checking it against `stg_stations`'s station name column.
4. Run `dbt test` and confirm every test you've added passes against your current data.
5. Deliberately break one test — for example, temporarily edit your `accepted_values` list to exclude a rider type you know exists in the data — run `dbt test` again, and confirm it fails with a clear message showing which rows violated it.
6. Revert your deliberate break from Task 5 and confirm `dbt test` passes again.
7. Add at least one `not_null` test to `stg_stations` and one to `stg_daily_weather`, choosing a column on each where a null value would genuinely indicate a data problem.

---

## Block 2: A Singular Custom Test (45 min)

**Goal:** Write a test that doesn't fit the four generic patterns, using a query that should return zero rows.

### Do this: 5 tasks

1. Write `tests/assert_trip_end_after_start.sql`, checking for any `stg_trips` row where `end_time` is not strictly after `start_time`.
2. Run `dbt test --select assert_trip_end_after_start` and confirm it passes (zero rows) or investigate any rows it flags.
3. Write a second singular test checking that no `fct_daily_rides` row has a negative `total_rides` value.
4. Write a third singular test of your own choosing, based on something you noticed while working with this data across the week — a specific data-quality concern you'd actually want caught automatically.
5. Run `dbt test` with no `--select` flag and confirm all generic and singular tests run together, with a clear pass/fail summary.

---

## Block 3: Documentation (60 min)

**Goal:** Write real descriptions for every model and key column, then generate and browse the resulting documentation site.

### Do this: 6 tasks

1. Add a `description` for every staging model and every mart model in their respective `schema.yml` files.
2. Add column-level `description`s for at least the primary/key columns of each model — the ones another person would need explained to trust the model at a glance.
3. Run `dbt docs generate` and `dbt docs serve`, and open the local documentation site in a browser.
4. Navigate to `fct_station_weather_volatility` in the docs site and trace its lineage graph backward to confirm it correctly shows all three staging models and the seed feeding into it.
5. Read through your own descriptions as if you'd never seen this project before, and rewrite at least one that assumes context a new reader wouldn't have.
6. Write a sentence on whether the lineage graph matched what you expected it to look like, or whether it revealed a dependency you hadn't consciously noticed.

---

## Block 4: A Reusable Macro (60 min)

**Goal:** Extract one piece of repeated logic into a macro, and use it in at least two different models.

### Do this: 6 tasks

1. Write `macros/pct_change.sql` following the pattern from the concepts file, accepting a value column and a previous-value column as arguments.
2. Use the macro inside `fct_daily_rides` to compute day-over-day percent change, replacing any inline `ROUND(... NULLIF(...))` expression you'd written there directly.
3. Use the same macro a second time in a different model — for example, computing week-over-week percent change in the same or a different mart — confirming the macro genuinely works with different column arguments, not just the one you first wrote it for.
4. Run `dbt run` and confirm both models still build correctly using the macro.
5. Compare the output of the macro-based percent change against a manually written version for a handful of rows, confirming they match exactly.
6. Write a sentence on how you'd decide, in general, whether a piece of repeated SQL logic is worth extracting into a macro versus just leaving it duplicated across two or three models.

---

## Block 5: An Incremental Model (60 min)

**Goal:** Convert your largest, ever-growing model to incremental materialization, and confirm it actually skips reprocessing old data on a second run.

### Do this: 6 tasks

1. Build `models/marts/fct_daily_rides_incremental.sql`, following the concepts file's pattern with `is_incremental()` and a `WHERE` filter against `{{ this }}`.
2. Run `dbt run --select fct_daily_rides_incremental` for the first time, and confirm it builds the full historical table, since `is_incremental()` is `false` on a first run.
3. Run it a second time immediately, with no new data added, and confirm — using your tool's timing if available — that this run is meaningfully faster than the first, since it should now be filtering to a date range with no new rows.
4. If you can simulate new data arriving (for example, temporarily filtering `stg_trips` to exclude the most recent week, running the incremental model, then removing that filter and re-running), do so, and confirm the incremental run correctly picks up only the new rows.
5. Run `dbt run --select fct_daily_rides_incremental --full-refresh` and confirm it rebuilds the entire table from scratch, ignoring the incremental filter — this is the escape hatch for when incremental logic needs to be corrected or the underlying data changed retroactively.
6. Write a sentence on why `unique_key` matters for this specific model — what would go wrong if you omitted it and a date's row count needed to be corrected after the fact?

---

## Block 6: Scheduling the Pipeline (45 min)

**Goal:** Ship a GitHub Actions workflow that runs the full dbt project on a schedule, without manual intervention.

### Do this: 6 tasks

1. Write `.github/workflows/dbt_build.yml` following the concepts file's pattern, running `dbt build` on a daily cron schedule.
2. Include `workflow_dispatch` so you can manually trigger a run from GitHub's Actions tab to test the workflow without waiting for the schedule.
3. Push the workflow file and trigger it manually from GitHub's UI, confirming the full pipeline — seed, run, test — completes successfully in the Actions log.
4. Deliberately introduce a failing test (the same kind of change as Block 1, Task 5) locally, push it, trigger the workflow again, and confirm the Actions run correctly fails and reports which test broke.
5. Revert the deliberate failure and confirm a subsequent workflow run passes again.
6. Write two or three sentences on what's still missing between this workflow and a production-grade pipeline at a real company — think about credentials, monitoring, alerting on failure, and how new raw data would actually get into the database before dbt runs against it.

---

## Block 7: Reflection (30 min)

Answer in a few sentences each:

1. Pick one test from Block 1 or Block 2 and describe a specific real scenario — not hypothetical — where it would have caught a problem in this dataset if the pipeline had been running unattended for a month.
2. Block 3 asked you to rewrite at least one description written for a stranger rather than for yourself. What made the original version insufficient, specifically?
3. Compare today's macro to the window functions from Days 1–2 and the recursive CTE from Day 3 — all three are ways of not repeating logic, at different levels (a SQL clause, a query, a Jinja snippet). Which of the three feels like the right tool for the trip-weather join from Day 2, and why not the other two?
4. Block 5's incremental model and Block 6's scheduled workflow solve two different problems that could easily be confused: one makes a single run faster, the other makes runs happen without you. Explain, in your own words, why a pipeline genuinely needs both, not just one or the other.
5. Looking back across the full week — window functions, running totals, recursive CTEs, query performance, pivoting, and now dbt — which single day's material do you expect to reach for most often in real work going forward, and which feels most like something you'll need to look up again before using?

**Deliverable:** The complete dbt project from Days 4–5, committed to GitHub: passing `dbt test` across all generic and singular tests, a generated and browsable `dbt docs` site, at least one macro used in two or more models, `fct_daily_rides_incremental` building correctly on both a full run and an incremental run, and `.github/workflows/dbt_build.yml` successfully running `dbt build` end to end from a manual trigger.