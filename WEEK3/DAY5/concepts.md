# Day 5 Concepts: Testing, Documentation & Turning dbt Into a Pipeline

**Estimated reading time: 55 min**

Yesterday built a working dbt project: sources, staging models, a seed, and mart models rebuilding several of this week's SQL files. Everything ran correctly, but nothing checked *whether* it kept running correctly, nothing explained the project to anyone reading it cold, and nothing ran on its own. Today covers all three — dbt's built-in testing and documentation, a first macro, an incremental model for a table that will keep growing, and a scheduled GitHub Actions workflow that runs the whole thing without you typing a command.

By the end of today, what you've built is a small but genuine data engineering pipeline: raw data in, tested and documented transformations, running on a schedule, with a full history in Git.

---

## Why Testing Matters Once Data Keeps Changing

Every query you've written this course was checked once, by eye, against a fixed dataset. A pipeline is different: new trip exports and weather observations arrive continuously, and nobody is manually re-checking every model every time. dbt's testing system exists to catch exactly the kind of problem that eye-checking a single run can't: a null value that shouldn't exist, a station ID in `trips` with no matching row in `stations`, a rider type that doesn't match any expected category.

## Generic Tests: Declared in YAML, Reused Everywhere

Most tests you'll write are **generic tests** — small, reusable checks declared against a column in a model's `schema.yml`, not written as SQL by hand each time. dbt ships four built in:

```yaml
# models/staging/schema.yml
version: 2

models:
  - name: stg_trips
    columns:
      - name: trip_id
        tests:
          - unique
          - not_null
      - name: rider_type
        tests:
          - accepted_values:
              values: ['member', 'casual']
      - name: start_station
        tests:
          - relationships:
              to: ref('stg_stations')
              field: station_name
```

- `not_null` — the column should never be null.
- `unique` — every value in the column should be distinct.
- `accepted_values` — the column should only ever contain one of a fixed list of values.
- `relationships` — every value in this column should exist in some column of another model, checking referential integrity the way a foreign key would in a transactional database.

`dbt test` runs every declared test and reports pass/fail per test, without you writing a single `SELECT` for any of them.

## Singular Tests: Custom SQL for Anything Generic Tests Can't Express

Some checks don't fit the four generic patterns. A **singular test** is just a `.sql` file in the `tests/` folder containing a query that should return **zero rows** if the data is correct — dbt treats any row the query does return as a failure:

```sql
-- tests/assert_trip_end_after_start.sql
SELECT trip_id
FROM {{ ref('stg_trips') }}
WHERE end_time <= start_time
```

If this query ever returns a row, something is wrong with that specific trip record, and `dbt test` fails on it. This is the same "write a query that finds the problem rows" instinct you've used all week for anomaly detection — a singular test is that same query, just wired into dbt's test runner instead of run by hand.

## Documentation: Descriptions That Become a Real Website

Descriptions written in `schema.yml` aren't just comments — `dbt docs generate` turns them into a browsable site with every model, every column, its description, and a full lineage graph showing exactly how data flows from source to mart:

```yaml
models:
  - name: fct_daily_rides
    description: >
      One row per calendar date, with total ride count, day-over-day
      change, and a trailing 7-day moving average. Built from stg_trips.
    columns:
      - name: calendar_date
        description: The date this row summarizes.
      - name: total_rides
        description: Count of trips starting on this date.
```

```bash
dbt docs generate    # builds the documentation site from your project + schema.yml files
dbt docs serve        # opens it locally in a browser
```

This matters more as a project grows than it seems to on a five-model project — the lineage graph is often the fastest way to answer "what would break if I changed this column," a question that gets genuinely hard to answer by reading SQL files alone once a project has dozens of models.

## Macros: Reusable Logic Written Once

A **macro** is a reusable snippet of Jinja (the templating language dbt SQL files are actually written in — every `{{ ref(...) }}` and `{{ source(...) }}` you've used so far is a macro call) that you define once and call from multiple models. The percent-change calculation from Day 2, repeated with slightly different columns across a few marts, is a reasonable first macro:

```sql
-- macros/pct_change.sql
{% macro pct_change(value_column, previous_column) %}
    ROUND(
      100.0 * ({{ value_column }} - {{ previous_column }})
      / NULLIF({{ previous_column }}, 0),
      1
    )
{% endmacro %}
```

Used inside a model:

```sql
SELECT
  calendar_date,
  total_rides,
  LAG(total_rides) OVER (ORDER BY calendar_date) AS previous_day_rides,
  {{ pct_change('total_rides', 'LAG(total_rides) OVER (ORDER BY calendar_date)') }} AS pct_change
FROM {{ ref('stg_trips_daily') }}
```

The payoff is the same one behind `ref()` and staging models: write the logic once, and every model that needs it calls the macro instead of re-typing the `ROUND(... NULLIF(...))` pattern and risking a small inconsistency between copies.

## Incremental Models: Stop Rebuilding What Hasn't Changed

`trips` only grows — new rides get appended, old ones never change. Rebuilding a `table`-materialized model from the *entire* history of `trips` on every single run is correct but wasteful once that history is large. An **incremental** model processes only new rows after the first run:

```sql
-- models/marts/fct_daily_rides_incremental.sql
{{
  config(
    materialized='incremental',
    unique_key='calendar_date'
  )
}}

SELECT
  DATE(start_time) AS calendar_date,
  COUNT(*) AS total_rides
FROM {{ ref('stg_trips') }}

{% if is_incremental() %}
  WHERE DATE(start_time) > (SELECT MAX(calendar_date) FROM {{ this }})
{% endif %}

GROUP BY calendar_date
```

`is_incremental()` is `false` on the very first run (or after a full `--full-refresh`), so the model builds from the complete `stg_trips` history once. On every run after that, it's `true`, and the `WHERE` clause restricts the query to only dates newer than whatever's already in the table — `{{ this }}` refers to the model's own already-built table, which is what makes the incremental filter possible. `unique_key` tells dbt how to handle a row that reappears (for example, if a date's count needs correcting after late-arriving trip data) — dbt updates the matching row instead of inserting a duplicate.

## From `dbt run` to a Scheduled Pipeline

Everything so far still requires you to type a command. A **pipeline**, in the data-engineering sense, runs on its own. The most direct way to get there for a project this size — without adopting a full orchestrator — is a scheduled GitHub Actions workflow, the same approach already used for the SB Finance Research stock-signal pipeline:

```yaml
# .github/workflows/dbt_build.yml
name: dbt build

on:
  schedule:
    - cron: '0 6 * * *'   # every day at 06:00 UTC
  workflow_dispatch: {}     # allows manually triggering a run from GitHub's UI

jobs:
  run-dbt:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install dbt-core dbt-duckdb
      - run: dbt build --profiles-dir .
```

`workflow_dispatch` is worth including even on a scheduled workflow — it adds a "Run now" button in GitHub's Actions tab, useful for testing the workflow itself without waiting for the cron trigger. Note that `profiles.yml` normally lives outside the repo for credential-safety reasons; for a project connecting only to a local DuckDB file with no real secrets, committing a profile (or generating one as a workflow step) is a reasonable simplification — a project connecting to a cloud warehouse would instead inject credentials as encrypted GitHub Actions secrets.

## A Brief Note on CI

A closely related pattern, common enough to be worth naming even without building it today: running `dbt build` automatically on every pull request, before it merges — usually against a temporary schema so it doesn't touch production data — so a broken model or a failing test is caught in review rather than after it's already scheduled to run. The GitHub Actions workflow above triggers on a schedule; the same mechanism, triggered `on: pull_request` instead, is what most teams mean by "dbt CI."

---

## Where This Leaves You

Put together, Days 4 and 5 form a genuinely complete, if small, data engineering pipeline: raw trip and weather data arrives (sources), a small reference table is version-controlled in the repo (seed), each raw table gets a single cleaned representation (staging), business logic gets built on top of that once (marts), every model is checked automatically (tests), the whole thing is documented and browsable (docs), and it runs on its own on a schedule (GitHub Actions) instead of waiting for someone to type `dbt run`. Nothing about this shape changes at a larger company — a production dbt project with hundreds of models is built from exactly these same pieces, just more of them.

## Learning Resources

A few resources worth working through after today, once the shape of a dbt project is no longer new:

- **Coursera — Analytics Engineering Workflows with dbt** — covers Jinja and macros, incremental models, snapshots, and Git-based review workflows in more depth than today's file: [coursera.org/learn/analytics-engineering-workflows-with-dbt](https://www.coursera.org/learn/analytics-engineering-workflows-with-dbt)
- **Coursera — Open Source Data Engineering with Spark, dbt & Airflow (Professional Certificate)** — the natural next step after today's GitHub Actions scheduling section, covering a real orchestrator (Airflow) triggering dbt runs instead of a cron-triggered Action: [coursera.org/professional-certificates/open-source-data-engineering-with-spark-dbt-and-airflow](https://www.coursera.org/professional-certificates/open-source-data-engineering-with-spark-dbt-and-airflow)
- **YouTube — [Data Build Tool] DBT: The Ultimate Guide, With CI/CD** — a longer, free walkthrough that goes further into CI/CD than today's brief note does: [youtube.com/watch?v=B8uwFmVt4sU](https://www.youtube.com/watch?v=B8uwFmVt4sU)
- **dbt-duckdb adapter documentation** — worth bookmarking directly, since it covers DuckDB-specific configuration (extensions, MotherDuck, multiple attached databases) not covered in dbt's general docs: [github.com/duckdb/dbt-duckdb](https://github.com/duckdb/dbt-duckdb)

## What's Next

This closes out Week 3. Today's task file adds tests and documentation to Thursday's models, writes a first macro, converts one model to incremental, and ships a scheduled GitHub Actions workflow — the deliverable for the week is the full project, tested, documented, and running on its own.