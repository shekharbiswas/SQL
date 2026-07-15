# SQL & Modern Data Tools

At Dataskools, we build data skills on real data, not toy examples. This six-week course takes learners from SQL fundamentals to modern data warehousing, progressing week by week from core querying through joins, data cleaning, advanced SQL, dashboarding, and modern data modeling with dbt and Snowflake.

Week 1 is grounded in a real dataset — Capital Bikeshare trip data (2019–2026, seven years) joined with NOAA weather data — to teach the fundamentals on something with genuine scale and messiness. By the end of Week 1, learners go from raw `SELECT` statements all the way to joining trips with station reference data and summarizing results. From Week 2 onward, the course moves through progressively more advanced SQL and data engineering concepts, applied to datasets appropriate to each topic.

## Course Structure

Six weeks, progressing from SQL basics through professional-level querying, dashboarding, and into modern data infrastructure — dbt for transformation, Snowflake for warehousing.

---

### Week 1: SQL Foundations — From First Query to First Join

**Goal:** Learn core SQL syntax, aggregate and summarize data, and combine tables — all against the same real dataset.

The Capital Bikeshare trip table is the starting point: millions of rows spanning 2019 to 2026, one row per ride. Learners move from raw filtering to a full joined, aggregated summary table.

Topics covered:
- What SQL is, and how relational data is structured
- `SELECT`, `FROM`, `WHERE`, `ORDER BY`, `LIMIT`
- Filtering by text, numeric, and date fields; working with timestamps across a multi-year range
- `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`
- `GROUP BY` with `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`
- `HAVING` versus `WHERE`; aliases and rounding
- Joining trips to a station reference table; `NULL` handling in joins

Mini-project: A joined, aggregated top-stations summary table — combining filtering, grouping, and joins into one reusable query.

---

### Week 2: Subqueries, CTEs & Data Cleaning

**Goal:** Write modular queries and handle the inconsistencies that come with real-world data.

Real datasets are rarely clean. Fields go missing, categories shift over time, and trends can be distorted by one-off events. This week is about handling that directly instead of ignoring it.

Topics covered:
- Subqueries in `SELECT`, `WHERE`, and `FROM`
- Common Table Expressions (CTEs)
- `NULL` handling, `COALESCE`, `CASE WHEN`
- Detecting and flagging anomalous periods in a time series

Mini-project: Identify a meaningful trend in the data, and separate genuine change from data gaps or one-off anomalies.

---

### Week 3: Advanced SQL & Intro to dbt

**Goal:** Write SQL at a professional data-engineering level, and get a first look at how modern data teams organize that SQL into a real pipeline.

This week pushes past application-level querying into the techniques used in production analytics work, then introduces the tool that structures it.

Topics covered:
- Window functions: `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, `LEAD`
- Running totals, moving averages, and period-over-period comparisons
- Recursive CTEs
- Query performance basics: reading `EXPLAIN` plans, indexing concepts
- Pivoting and unpivoting data
- What dbt is, why data teams use it, and how it differs from writing raw SQL

Mini-project: Rewrite an earlier query using window functions to solve something a `GROUP BY` alone couldn't.

---

### Week 4: SQL + Power BI Project, and Intro to Snowflake

**Goal:** Turn advanced SQL into a real dashboard, and get a first look at cloud data warehousing.

Topics covered:
- Connecting Power BI to a SQL source
- Building dashboards from professional-level SQL output: bar and line charts, slicers
- What a cloud data warehouse is and why teams use one
- Snowflake's basic architecture: separation of storage and compute

Deliverables:
- Power BI file (`.pbix`)
- Dashboard screenshot
- Short written reflection on the build

Estimated time: about eight hours.

---

### Week 5: dbt Deep Dive

**Goal:** Move from standalone queries to a modeled, testable transformation pipeline.

By this point the SQL has grown in number and complexity. This week restructures that work properly using dbt.

Topics covered:
- dbt project structure: sources, staging models, intermediate models, marts
- DAGs and model lineage
- dbt tests: not null, uniqueness, relationships, accepted values
- Documentation generation
- Incremental models for a growing multi-year dataset

Activities:
- Build staging models for the underlying source tables
- Build an intermediate model joining the staged sources
- Add tests and generate documentation
- Review the resulting DAG

---

### Week 6: Snowflake Deep Dive

**Goal:** Move the pipeline into a real cloud data warehouse.

Topics covered:
- Snowflake architecture in depth: virtual warehouses, storage/compute separation, scaling
- Loading data into Snowflake
- Roles, permissions, and access basics
- Query performance and cost/credit management concepts
- Connecting dbt to Snowflake and running the pipeline end to end

Deliverables:
- SQL scripts and dbt project connected to and running on Snowflake
- Short written report: three data-backed observations and one recommendation based on the findings

Wrap-up: GitHub portfolio setup, an overview of data analytics career tracks, and course completion review.

---

## Concepts Introduced Along the Way

Beyond the core weekly topics, learners are gradually introduced to ideas used by real data teams:

- **Materialized views** — caching expensive queries to speed up dashboards
- **Dimensional modeling** — star and snowflake schema design
- **Incremental models** — handling a growing multi-year dataset without reprocessing everything each time
- **Data quality testing** — catching gaps and anomalies before they reach a dashboard

## What You'll Have Built by the End

A working, end-to-end pipeline: professional-level SQL, a Power BI dashboard, a modeled dbt project, and that pipeline running live on Snowflake — built progressively over six weeks.