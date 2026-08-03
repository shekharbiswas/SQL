# Day 4 Task: Setting Up dbt and Building the First Models

**Total time: 6 hours**

Today moves the bikeshare and weather data you've been querying by hand into an actual dbt project: a real project folder, sources declared against the raw tables, a seed for the weather-station lookup, and a first set of staging and mart models — including a dbt version of Day 3's date-spine recursive CTE and Day 2's trip-weather volatility work.

Read `week3_day4_concepts.md` before starting if you haven't already — this task file assumes you understand `source()` vs. `ref()`, the staging/marts convention, and the difference between `view`, `table`, and `incremental` materializations.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Environment setup and connecting to DuckDB | 60 min |
| 2 | Declaring sources | 45 min |
| 3 | Staging models | 90 min |
| 4 | A seed for the weather-station lookup | 45 min |
| 5 | Mart models | 90 min |
| 6 | Reflection | 30 min |

---

## Block 1: Environment Setup and Connecting to DuckDB (60 min)

**Goal:** Get a working dbt project pointed at the same DuckDB database you've used all week, with a passing `dbt debug`.

### Do this: 6 tasks

1. Create and activate a Python virtual environment, then install `dbt-core` and `dbt-duckdb`.
2. Run `dbt init capital_bikeshare` (or your preferred project name) to scaffold a new project, and look through the generated folder structure before changing anything.
3. Set up `~/.dbt/profiles.yml` pointing `path` at your existing bikeshare DuckDB file from this course, rather than creating a new empty database.
4. Run `dbt debug` and confirm every check passes. If the connection fails, resolve it before continuing — don't move on with a broken connection.
5. Run `dbt run` against the empty starter project (it should build only dbt's example models, if any were scaffolded) just to confirm the full command works end to end.
6. Delete or ignore any example models dbt scaffolded for you, and confirm your `models/` folder is clean before Block 2.

---

## Block 2: Declaring Sources (45 min)

**Goal:** Name the raw tables you didn't build with dbt, so they become real nodes in the dependency graph.

### Do this: 5 tasks

1. Create `models/staging/sources.yml` and declare `trips`, `stations`, and `daily_weather` as tables under a single source named `raw_bikeshare`.
2. Confirm the schema/database name in your source YAML actually matches where these tables live in your DuckDB file — check with a direct query if you're not sure.
3. Write a throwaway model that selects from `{{ source('raw_bikeshare', 'trips') }}` with no other logic, run it, and confirm it returns the same row count as querying `trips` directly.
4. Run `dbt ls --resource-type source` (or your dbt version's equivalent) to confirm dbt recognizes all three declared sources.
5. Delete the throwaway model from Task 3 once you've confirmed sources resolve correctly — it isn't part of the real project.

---

## Block 3: Staging Models (90 min)

**Goal:** Build one staging model per raw source, following the one-to-one, lightly-cleaned convention from the concepts file.

### Do this: 7 tasks

1. Build `models/staging/stg_trips.sql`, selecting from `{{ source('raw_bikeshare', 'trips') }}` with consistent column names and a filter removing any row with a null `start_time`.
2. Build `models/staging/stg_stations.sql` from the `stations` source, with consistent naming.
3. Build `models/staging/stg_daily_weather.sql` from the `daily_weather` source, with consistent naming.
4. Run `dbt run --select staging.*` (or `--select models/staging`, depending on your dbt version) and confirm all three staging models build successfully as views.
5. Query one staging model directly in Beekeeper Studio (or your SQL client) and compare its row count against the raw source table it was built from — confirm your filtering in Task 1 removed the expected number of rows, and no more.
6. Add a short `description` for each staging model and its columns in a `models/staging/schema.yml` file — you won't test them until Day 5, but write the descriptions now while the cleaning logic is fresh.
7. Write a sentence on what, if anything, felt different about writing `SELECT ... FROM {{ source(...) }}` compared to a plain `SELECT ... FROM trips` you'd have written any other day this week.

---

## Block 4: A Seed for the Weather-Station Lookup (45 min)

**Goal:** Load the bike-station-to-weather-station mapping as a seed, and confirm it behaves like any other referenceable model.

### Do this: 5 tasks

1. Create `seeds/weather_station_lookup.csv` with one row per bike station and its `nearest_weather_station_id`, matching the mapping you used for Week 3, Day 2's trip-weather join.
2. Run `dbt seed` and confirm the CSV was loaded into the database as a table.
3. Query the resulting table directly and confirm the row count matches the number of lines in your CSV (minus the header).
4. Reference the seed from `stg_stations.sql` using `{{ ref('weather_station_lookup') }}`, joining it in so `stg_stations` carries `nearest_weather_station_id` going forward.
5. Re-run `dbt run --select stg_stations` and confirm the join resolved correctly — every station should have a non-null `nearest_weather_station_id`, assuming your CSV covers every station.

---

## Block 5: Mart Models (90 min)

**Goal:** Rebuild several of this week's SQL files as dbt mart models, reading only from staging models via `ref()`.

### Do this: 7 tasks

1. Build `models/marts/dim_date_spine.sql`, porting over Day 3's recursive `WITH RECURSIVE` date-spine query as a standalone model with no `ref()` dependency (it doesn't need one — it generates dates from scratch).
2. Build `models/marts/fct_daily_rides.sql`, aggregating `{{ ref('stg_trips') }}` into one row per date with `total_rides`, plus the `LAG()`-based day-over-day change from Day 1 and the trailing 7-day average from Day 2.
3. Build `models/marts/fct_station_weather_volatility.sql`, joining `{{ ref('stg_trips') }}`, `{{ ref('stg_stations') }}`, and `{{ ref('stg_daily_weather') }}` (using the `nearest_weather_station_id` now available on `stg_stations` from Block 4), and porting over Day 2's trailing `STDDEV()` volatility columns.
4. Add `{{ config(materialized='table') }}` to the top of `fct_daily_rides.sql` and `fct_station_weather_volatility.sql`, since both are read repeatedly and don't need to be recomputed on every query.
5. Run `dbt run` for the full project and confirm dbt builds every model in the correct dependency order without you specifying it manually.
6. Run `dbt docs generate` and, without worrying about the details yet (Day 5 covers documentation properly), open the generated DAG view and confirm it visually matches the dependency structure you expect: sources → staging → marts.
7. Query `fct_daily_rides` and `fct_station_weather_volatility` directly and spot-check a handful of values against the equivalent standalone queries you wrote in Days 1–2, confirming the numbers match.

---

## Block 6: Reflection (30 min)

Answer in a few sentences each:

1. Pick one model you built today and describe, specifically, what would have to change in your workflow if you'd written the same logic as a standalone `.sql` file instead — not just "it would be harder," but what concretely would you lose.
2. Block 4 turned the weather-station lookup into a seed rather than a source. Write a sentence on how you'd decide, in general, whether a piece of reference data belongs as a seed (in the repo) versus a source (pulled from somewhere else) versus a hardcoded value inside a model.
3. Block 5 asked you to mark two mart models as `table` materializations. Pick one of your staging models and decide whether it should stay a `view` or become a `table` — what's the actual trade-off you're weighing?
4. `dim_date_spine` was the one model today with no `ref()` dependency at all. Does that make it a source, a staging model, or something else in the staging/marts convention — or does it not fit cleanly into either category?
5. Tomorrow adds tests, documentation, a macro, an incremental model, and a scheduled GitHub Actions workflow. Based on what you built today, which model do you expect will benefit most from becoming incremental, and why?

**Deliverable:** Your dbt project committed to GitHub, including `sources.yml`, the three staging models, the `weather_station_lookup` seed, and the three mart models (`dim_date_spine`, `fct_daily_rides`, `fct_station_weather_volatility`) — with `dbt run` completing successfully end to end from a clean checkout.