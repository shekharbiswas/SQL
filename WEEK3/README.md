# Overview: From Trustworthy Queries to a Production Pipeline

## Where We're Picking This Up

Week 2 closed with a repo that finally held together: subqueries and CTEs for multi-step questions, `NULL` handling that actually fixed the station gap instead of working around it, and a defended anomaly finding built on a clean daily time series. That was enough to trust a single number — but every query that got us there was still standalone. The trailing-average calculation was a correlated subquery re-scanning the same window for every row. The period-over-period comparison was a self-join. Nothing was reusable, nothing was tested, and nothing ran without a person opening Beekeeper Studio and clicking run.

Week 3 is about closing those loops. We move from SQL that's individually correct to SQL that's efficient, organized, and eventually running on its own — ending the week with a real, if small, data pipeline instead of a folder of scripts.

## Why This Week, and Why Now

This week has a clear arc, and each day exists because the day before it created a need:

- **Window functions (Day 1)** replace the self-joins and correlated subqueries from Week 2, Day 4 with a tool built specifically for "compare this row to nearby rows" — the exact question that made those patterns feel like more machinery than the question deserved.
- **Window frames, running totals & volatility (Day 2)** extend Day 1's `OVER()` clause to a specific range of rows, finishing the trailing-average replacement for real, and introduce a new weather dataset to measure spread, not just center — because averages alone couldn't tell us whether conditions near a station had been stable or swinging.
- **Recursive CTEs, query performance & pivoting (Day 3)** exist because three things were still missing that don't fit the "compare nearby rows" theme at all: the date spine every earlier query quietly depended on had to come from somewhere, nobody had actually looked at *why* one query beats another, and reshaping data between long and wide form kept coming up without a clean answer.
- **dbt fundamentals (Day 4)** exist because by this point, several queries — the trip-weather join especially — were being rewritten in multiple places, with no dependency tracking and no way to tell what would break if a raw column changed. dbt is the tool that organizes exactly that.
- **dbt testing, documentation & scheduling (Day 5)** exist because a dbt project that isn't tested, documented, or running on its own is still just a well-organized folder of scripts, not a pipeline. Day 5 is what turns Day 4's project into something that checks itself and runs without a person typing a command.

So the concrete problem for the week is:

> "We can write SQL that answers hard questions correctly, but every answer still lives in a standalone file, gets recomputed the expensive way, and depends on us remembering to run it. Can we get to a point where our transformations are organized, efficient, tested, and running on their own?"

## Weekly Breakdown

| Day | Theme | Discovery Focus |
|-----|-------|------------------|
| 1 | Window Functions | Replace Week 2's self-joins and correlated subqueries with `OVER()`, `PARTITION BY`, `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, and `LEAD` — and feel where a window function is doing in one line what used to take a join. |
| 2 | Window Frames, Running Totals & Volatility | Extend `OVER()` with explicit frames for running totals and moving averages, replace the Week 2 trailing-average correlated subquery for good, and bring in a weather dataset to measure volatility with `STDDEV()`, not just center with `AVG()`. |
| 3 | Recursive CTEs, Query Performance & Pivoting | Build the date spine that's been quietly powering the week's daily series, read `EXPLAIN` plans to understand *why* a query is fast or slow, and reshape data both directions with `PIVOT`/`UNPIVOT`. |
| 4 | dbt Fundamentals | Set up a real dbt project against the bikeshare database: sources, staging models, a seed, and marts rebuilding the week's window-function and recursive-CTE work with dependency tracking via `ref()`. |
| 5 | dbt Testing, Docs & Scheduling | Add generic and singular tests, generate real documentation and a lineage graph, extract a macro, convert a growing model to incremental, and ship a GitHub Actions workflow that runs the whole project on a schedule. |

Below is a closer look at what each day actually involves.

**Day 1 — Window Functions**
The day starts from Week 2, Day 4's specific complaint — a correlated subquery re-scanning rows, a self-join doing more work than the question needed — and introduces `OVER()`, the empty-window case, `PARTITION BY` for scoping without collapsing rows, `ROW_NUMBER`/`RANK`/`DENSE_RANK` for ranking (and where the two tie-handling functions genuinely diverge), and `LAG`/`LEAD` for comparing a row to its neighbors without a join. Several Week 1 and Week 2 answers get rebuilt side by side with their new window-function equivalents, so the improvement isn't just asserted, it's checked.

**Day 2 — Window Frames, Running Totals & Volatility**
With `OVER()` and `PARTITION BY` established, this day adds the piece that makes running totals and moving averages possible: the frame, and the default-frame trap that silently turns an intended overall average into a running one. The trailing 7-day average from Week 2, Day 4 gets replaced with a real frame clause, `ROWS` and `RANGE` get told apart on data that actually has ties, and a new weather dataset — joined to trips by event location, not just date — introduces `STDDEV()` as a window function, measuring how volatile conditions or ridership have been, not just their average.

**Day 3 — Recursive CTEs, Query Performance & Pivoting**
This day steps outside the "compare nearby rows" theme entirely. `WITH RECURSIVE` builds a gap-free date spine from scratch — the same one every daily series this week has secretly depended on — and follows a single bike across a chain of trips, a genuinely hierarchical relationship no fixed number of joins could handle. `EXPLAIN` output gets read for real, distinguishing a sequential scan from an index scan and connecting query performance concepts to DuckDB's columnar, vectorized engine specifically. `PIVOT` and `UNPIVOT` handle reshaping between long and wide data in both directions, closing out the toolkit before dbt changes the subject.

**Day 4 — dbt Fundamentals**
This is where standalone SQL files become an organized project. Sources declare the raw tables dbt didn't build; staging models give each one a single, lightly-cleaned representation; a seed brings the weather-station lookup into version control; and marts — `dim_date_spine`, `fct_daily_rides`, `fct_station_weather_volatility` — rebuild the week's actual window-function and recursive-CTE work, wired together with `ref()` so dbt tracks what depends on what automatically.

**Day 5 — dbt Testing, Docs & Scheduling**
The final day makes yesterday's project trustworthy and unattended. Generic tests (`not_null`, `unique`, `accepted_values`, `relationships`) and a singular custom test catch data problems automatically instead of by eye; `dbt docs generate` turns descriptions into a browsable lineage graph; a macro extracts the percent-change pattern instead of repeating it; an incremental model stops rebuilding all of `trips` on every run; and a scheduled GitHub Actions workflow runs the whole thing on a cron trigger, closing the loop from "a project we run by hand" to "a pipeline that runs on its own."

## Learning Objectives

By the end of this week, learners will be able to:

- Write and choose between `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, and `LEAD`, and explain what each replaces from Week 1–2's subquery and self-join patterns
- Use `PARTITION BY` to scope a window function per group without collapsing rows, and combine multiple partitioned window functions in one query
- Define an explicit window frame with `ROWS BETWEEN`, recognize the default-frame trap, and tell `ROWS` and `RANGE` apart on data with genuine ties
- Build running totals, trailing and centered moving averages, and period-over-period percent change using window functions instead of correlated subqueries or self-joins
- Use `STDDEV()` (and related aggregates) as a window function to measure volatility, and join two datasets at the event-location level to support that analysis
- Write a recursive CTE with a correct anchor, recursive member, and terminating condition, and recognize when a problem's shape — an unknown-depth chain or a self-referencing structure — actually calls for one
- Read `EXPLAIN` output well enough to distinguish a sequential scan from an index scan, and reason about indexing trade-offs in both a traditional row-store and a columnar engine like DuckDB
- Reshape data between long and wide form using `PIVOT` and `UNPIVOT`, and recognize which direction a given reporting or ingestion problem actually needs
- Explain what dbt does that a folder of SQL files doesn't — dependency tracking via `ref()`, sources, testing, and documentation — and set up a working dbt project against DuckDB
- Structure a dbt project using the staging/marts convention, choose an appropriate materialization per model, and use a seed for small, version-controlled reference data
- Write generic and singular dbt tests, generate project documentation, extract a macro, build an incremental model, and schedule a dbt project to run unattended via GitHub Actions

## Difficulty Snapshot — Week 3

Where Week 2 was about combining queries and trusting the data underneath them, Week 3 is about efficiency, structure, and eventually handing transformation logic off to a system that runs it for you.

| Topic Area | Difficulty | What to Expect |
|------------|------------|-----------------|
| OVER(), PARTITION BY, ROW_NUMBER | Easy–Medium | The concept clicks quickly once you see a window function keep every row instead of collapsing them; the syntax itself isn't the hard part. |
| RANK vs. DENSE_RANK, LAG/LEAD | Medium | Straightforward mechanically, but choosing the *right* function for a given report (gapped ranks vs. dense ranks; which row counts as "previous") takes real judgment. |
| Window frames & the default-frame trap | Hard | The default frame silently changing `AVG()` into a running average instead of an overall one is the first genuinely easy-to-miss bug this week — the kind that produces a plausible, wrong number. |
| STDDEV() and the weather join | Medium | The window-function mechanics are identical to `AVG()`; the harder part is the join itself, matching trips to weather by both date and event location correctly. |
| Recursive CTEs | Hard | The anchor/recursive-member structure is unfamiliar, and a broken terminating condition is a genuinely different failure mode than anything seen so far — it doesn't return a wrong answer, it can just keep running. |
| EXPLAIN & indexing | Medium | Reading a plan is approachable once you know what to look for; reasoning correctly about DuckDB's columnar engine versus a traditional row-store takes more careful thought than the syntax does. |
| PIVOT / UNPIVOT | Easy–Medium | The shape-shifting concept is intuitive; the harder judgment call is recognizing when raw source data needs unpivoting *before* anything else in the toolkit can touch it. |
| dbt setup: sources, staging, ref() | Medium | The ideas are simple once demonstrated, but getting `profiles.yml`, the DuckDB adapter, and folder conventions right the first time takes some trial and error. |
| dbt marts & materializations | Medium | Mostly a matter of porting logic you already trust; deciding `view` vs. `table` vs. `incremental` per model is a design judgment, not a syntax one. |
| dbt tests, docs, macros | Easy–Medium | Declarative and approachable once the pattern is seen once; the discipline of writing a good description or deciding what's worth a macro is the real skill. |
| Incremental models & scheduling | Hard | `is_incremental()` and `{{ this }}` are genuinely new mental models, and a scheduled GitHub Actions workflow is the first time this course's SQL runs completely unattended — worth testing carefully before trusting it. |

Tip: This week's hardest moments aren't syntax errors either — they're the moments where something runs without complaint but does the wrong thing quietly, like a default frame producing a running average instead of an overall one, or a recursive CTE that never terminates because the `WHERE` condition was slightly off. The habit worth building this week is checking the *shape* of a result against what you expected, not just whether the query ran.

## Tools Used This Week

- **DuckDB**, queried through **Beekeeper Studio** — the same setup from Weeks 1–2, now used for window functions, recursive CTEs, and `EXPLAIN` plans against the full seven-year `trips` table.
- **`stations` table** and a new **`daily_weather` table** — joined at the event-location level (each station's nearest weather station) for Day 2's volatility work.
- **dbt Core with the `dbt-duckdb` adapter** — introduced Day 4, used to organize sources, staging models, seeds, and marts against the same DuckDB database used all week.
- **GitHub Actions** — used Day 5 to schedule `dbt build` on a cron trigger, the same approach already used for the SB Finance Research stock-signal pipeline.
- **Git / GitHub repo from Weeks 1–2** — extended with a `sql/` folder for the week's standalone queries and a full dbt project folder (`models/`, `seeds/`, `macros/`, `tests/`, `.github/workflows/`) by Day 5.

## End-of-Week Outcomes

By the end of the week, learners should be able to:

- Take a query that used to require a self-join or correlated subquery and rebuild it as a window function, explaining specifically what's gone and what's gained
- Build a trailing moving average and a trailing volatility measure over the same data, and explain why they answer genuinely different questions
- Generate a gap-free date spine from scratch with a recursive CTE, and use it to confirm — rather than assume — that a daily time series has no missing dates
- Read an `EXPLAIN` plan well enough to say, in plain language, why one version of a query is faster than another
- Point to a working dbt project — sources, staging, marts, seeds, tests, docs — built from this week's own SQL, and explain what each layer is responsible for
- Open a GitHub Actions run and show a scheduled `dbt build` completing successfully, including a passing test suite, without having typed a command that day

This week's deliverable isn't a single finding the way Week 2's anomaly investigation was — it's a working pipeline: efficient, tested, documented SQL organized into a dbt project, running on its own schedule, with a full history in Git.

Reflection prompt: Compare this week's dbt project to Week 2's consolidated repo. Week 2 asked "does this repo hold together as one coherent project?" — Week 3 asks "does this project run itself, and would we trust it to?" Where did that shift change how you thought about a query while writing it, and where do you think it'll matter most once Week 4 puts a real dashboard and a cloud warehouse on top of what you built here?