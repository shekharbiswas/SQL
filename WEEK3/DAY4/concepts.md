# Day 4 Concepts: Introduction to dbt — From Standalone Queries to a Pipeline

**Estimated reading time: 55 min**

Every SQL file you've written this course has been a standalone script: open it, run it, read the result. That works fine at the scale of one query, but Week 3, Day 2 already hit its edge — the trip-weather join had to be repeated across several queries, and there was no way to tell, just by looking at a file, what it depended on or what would break if you changed it. **dbt** (data build tool) is what real data teams use to solve exactly that problem. Today covers what dbt actually is and how a project is structured; Day 5 covers testing, documentation, and turning a dbt project into something that runs on a schedule rather than by hand.

Everything today applies directly to the Capital Bikeshare and weather data you've been querying all week — by the end of Day 5, several of the SQL files you've already written will exist as dbt models in a working project.

---

## What dbt Actually Is

dbt does one specific job: it takes a folder of `SELECT` statements, figures out how they depend on each other, and runs them in the right order against your database — turning each one into a view or a table. That's the entire core idea. dbt doesn't extract data from source systems and it doesn't load data into your warehouse; those are the "E" and "L" in ELT, typically handled by something else (an export job, a sync tool, a script). dbt is the "T" — transformation — and it assumes the raw data is already sitting in your database before it starts.

What dbt adds on top of "a folder of SQL files" is what makes it worth learning:

- **Dependency tracking.** Instead of hardcoding a table name in a `FROM` clause, you reference another model by name, and dbt builds a full dependency graph (a DAG) automatically — it knows what has to run before what.
- **Version control by default.** A dbt project is just files on disk, meant to live in Git, exactly like the SQL files you've been committing all week.
- **Built-in testing.** You can declare "this column should never be null" or "this value should always match one of these five options" directly against a model, and dbt checks it every run.
- **Built-in documentation.** Descriptions you write for a model or column turn into a browsable, linked website showing exactly how data flows from raw source to final output.

None of this replaces the SQL you already know. A dbt model *is* a `SELECT` statement — the same kind you've written all week. dbt's job is organizing many of them together.

## Setting Up dbt Against DuckDB

This course has used DuckDB via Beekeeper Studio all week. dbt connects to DuckDB through a community-maintained adapter, `dbt-duckdb`, which installs both dbt itself and the DuckDB driver:

```bash
python -m venv dbt-env
source dbt-env/bin/activate      # on Windows: dbt-env\Scripts\activate
pip install dbt-core dbt-duckdb
```

dbt needs two things to run: a **project** (the folder of models, in your Git repo) and a **profile** (connection details, kept outside the repo since it can contain credentials — for DuckDB it's just a file path, but the convention still applies). The profile lives in `~/.dbt/profiles.yml`:

```yaml
capital_bikeshare:
  target: dev
  outputs:
    dev:
      type: duckdb
      path: 'capital_bikeshare.duckdb'
      threads: 4
```

`dbt debug` checks that dbt can find your project and successfully open a connection using this profile — run it right after setup, before writing a single model, so any connection problem is obvious immediately rather than buried under a confusing model-run error later.

## Anatomy of a dbt Project

`dbt init capital_bikeshare` scaffolds a starter project. The pieces that matter this week:

```
capital_bikeshare/
├── dbt_project.yml       -- project-level config
├── models/
│   ├── staging/           -- one model per raw source, lightly cleaned
│   └── marts/              -- business-level models, joined and aggregated
├── seeds/                 -- small reference CSVs dbt loads as tables
├── macros/                -- reusable Jinja/SQL snippets
└── tests/                 -- standalone custom tests (more on Day 5)
```

`dbt_project.yml` declares the project name and, importantly, default **materializations** per folder — more on that below.

## Sources: Naming the Raw Data You Didn't Build

Your `trips`, `stations`, and `daily_weather` tables already exist in the database — they're raw data, not something dbt created. Rather than typing those table names directly into a model's `FROM` clause, dbt has you declare them once as **sources**, in a YAML file:

```yaml
# models/staging/sources.yml
version: 2

sources:
  - name: raw_bikeshare
    schema: main
    tables:
      - name: trips
      - name: stations
      - name: daily_weather
      - name: weather_station_lookup
```

Any model can then reference `trips` as `{{ source('raw_bikeshare', 'trips') }}` instead of writing the table name literally. This looks like extra ceremony for one table, but it pays off once you have many models: declaring sources means dbt's dependency graph includes raw data as a real starting node, and if a source table's name ever changes, you fix it in one YAML file instead of hunting through every model that references it.

## Models and `ref()`: Building the Dependency Graph

A dbt **model** is a `.sql` file containing exactly one `SELECT` statement — no `CREATE TABLE`, no semicolon-separated statements. The filename becomes the model's name. Here's a first staging model, cleaning up the raw `trips` source:

```sql
-- models/staging/stg_trips.sql
SELECT
  trip_id,
  bike_id,
  rider_type,
  start_station,
  end_station,
  start_time,
  end_time
FROM {{ source('raw_bikeshare', 'trips') }}
WHERE start_time IS NOT NULL
```

Once `stg_trips` exists, any *other* model that needs trip data references it with `{{ ref('stg_trips') }}` instead of naming a raw table:

```sql
-- models/marts/fct_daily_rides.sql
SELECT
  DATE(start_time) AS calendar_date,
  COUNT(*) AS total_rides
FROM {{ ref('stg_trips') }}
GROUP BY calendar_date
```

`ref()` is the mechanism that makes the DAG real: dbt parses every model, sees which ones call `ref()` on which others, and knows automatically that `stg_trips` has to run before `fct_daily_rides`. You never write `dbt run stg_trips then dbt run fct_daily_rides` — you run the project, and dbt works out the order.

## Staging vs. Marts: A Layering Convention, Not a Rule dbt Enforces

`models/staging/` and `models/marts/` are a naming *convention* widely used across the dbt community, not something dbt requires. The idea behind it is worth understanding on its own terms:

- **Staging models** are close to one-to-one with a raw source: light cleaning, consistent column names, maybe a type cast — one staging model per source table, and nothing joined together yet. `stg_trips`, `stg_stations`, `stg_daily_weather` are staging models.
- **Marts models** are where the actual business logic lives: joins, aggregations, the window functions from Days 1–2, the recursive date spine from Day 3. Marts read from staging models (via `ref()`), never directly from sources.

The payoff of this separation is the same reasoning behind writing a source in one place: if a raw column ever gets renamed upstream, exactly one staging model needs to change, and every mart built on top of it keeps working untouched.

## Materializations: What a Model Actually Becomes

By default, dbt creates every model as a **view** — a saved query, re-run every time something selects from it. You can override this per model with a `config()` call at the top of the file, or in `dbt_project.yml` for a whole folder:

```sql
{{ config(materialized='table') }}

SELECT ...
```

- `view` (default): cheap to create, always reflects current source data, but re-computes the full query on every read — fine for staging models on a dataset this size.
- `table`: computed once at `dbt run` time and stored as an actual table — faster to query afterward, but stale until the next run. A good fit for a mart like `fct_daily_rides` that gets queried repeatedly.
- `incremental`: builds a table but only processes *new* rows on subsequent runs, instead of recomputing everything. This matters once a table is large enough that a full rebuild is wasteful — Day 5 covers it properly.

## Seeds: Small Reference Data That Lives in the Repo

Not everything belongs in a source table pulled from a raw data pipeline. The mapping from each bike station to its nearest weather station — the one Day 2 relied on for the trip-weather join — is exactly the kind of small, mostly-static lookup table that's easier to maintain as a plain CSV committed to Git than to treat as a "real" source:

```
seeds/weather_station_lookup.csv
```

```csv
station_name,nearest_weather_station_id
14th & Irving St NW,USW00013743
Pentagon City,USW00093738
...
```

`dbt seed` loads this CSV into the database as a table, which staging or mart models can then reference with `{{ ref('weather_station_lookup') }}` — the same `ref()` syntax as any other model, since to the rest of the project, a seed behaves just like one.

## Running the Project

A few commands cover almost everything you'll need this week:

```bash
dbt debug          # confirm the connection works
dbt seed           # load CSV seeds into the database
dbt run            # build every model, in dependency order
dbt run --select stg_trips+   # build stg_trips and everything downstream of it
dbt build          # seed + run + test together, in dependency order
```

`dbt build` is generally what you actually want day to day, once tests exist (Day 5) — it's the single command that loads seeds, builds models, and checks tests, stopping downstream work if something upstream fails.

## Where "Pipeline" Comes In

Running `dbt run` by hand from your terminal is exactly what you'll do all of today — but it's not yet a *pipeline* in the data-engineering sense, because nothing is triggering it automatically. A real pipeline needs raw data to keep arriving (new trip exports, new weather observations) and the transformation step to keep re-running on some schedule without a person typing the command each time. dbt itself doesn't schedule anything; it's typically paired with an orchestrator — dbt Cloud's own scheduler, a tool like Airflow, or, for a project this size, a scheduled GitHub Actions workflow that checks out the repo and runs `dbt build` on a cron trigger. Day 5 builds exactly that, using the same GitHub Actions approach already used for the SB Finance Research stock-signal pipeline.

---

## Learning Resources

Today and tomorrow cover enough dbt to work independently on this project, but dbt is a large enough tool that outside structured learning is worth doing in parallel rather than after:

- **dbt Labs' own free course** — "dbt Fundamentals," built and maintained by the team that created dbt, roughly 5 hours end to end: [learn.getdbt.com/courses/dbt-fundamentals](https://learn.getdbt.com/courses/dbt-fundamentals)
- **Coursera — Analytics Engineering with dbt (Specialization)** — three courses covering the modern data stack, dimensional modeling, dbt development, testing, and CI/CD, with a capstone project: [coursera.org/specializations/analytics-engineering-with-dbt](https://www.coursera.org/specializations/analytics-engineering-with-dbt) — worth using the Coursera Plus access for this one directly.
- **Coursera — Open Source Data Engineering with Spark, dbt & Airflow (Professional Certificate)** — broader than just dbt, but useful once Day 5's orchestration section raises the question of where dbt fits next to a real scheduler: [coursera.org/professional-certificates/open-source-data-engineering-with-spark-dbt-and-airflow](https://www.coursera.org/professional-certificates/open-source-data-engineering-with-spark-dbt-and-airflow)
- **YouTube — dbt Crash Course for Beginners (dbt Core)** — a focused, free walkthrough of the same install-project-model-run flow this file just covered: [youtube.com/watch?v=toSAAgLUHuk](https://www.youtube.com/watch?v=toSAAgLUHuk)
- **YouTube — Zero to dbt (dbt Labs' own channel)** — a fast-paced official walkthrough of the fundamentals, useful as a second pass after today's reading: [youtube.com/watch?v=a3fRALauYWs](https://www.youtube.com/watch?v=a3fRALauYWs)

None of these are required to complete today's task file, but the dbt Labs Fundamentals course in particular is worth doing this week rather than later — it uses dbt Cloud rather than dbt Core, so seeing both interfaces side by side is useful before you commit to one workflow.

## What's Next

Today's task file sets up a real dbt project against your existing bikeshare database, declares sources, builds a first layer of staging models, adds the weather-station lookup as a seed, and rebuilds several of Week 3's window-function and recursive-CTE queries as mart models. Day 5 adds tests, documentation, a macro, an incremental model, and a scheduled GitHub Actions workflow — turning what's built today from a project you run by hand into something closer to a real, unattended pipeline.