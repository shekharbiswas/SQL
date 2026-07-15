# SQL & Modern Data Tools

At Dataskools, we build data skills on real data, not toy examples. This six-week course takes learners from SQL fundamentals to modern data warehousing, progressing week by week from core querying through joins, data cleaning, dashboarding, and modern data modeling with dbt and Snowflake.

Week 1 is grounded in a real dataset — Capital Bikeshare trip data (2019–2026, seven years) joined with NOAA weather data — to teach the fundamentals on something with genuine scale and messiness. From Week 2 onward, the course moves through progressively more advanced SQL and data engineering concepts, applied to datasets appropriate to each topic.

## Course Structure

Six weeks, progressing from SQL basics through dbt and Snowflake, with Power BI as the visualization layer once the SQL foundation is solid.

---

### Week 1: SQL Foundations — Reading the Ride Data

**Goal:** Learn core SQL syntax and get comfortable extracting information from raw trip records.

The Capital Bikeshare trip table is the starting point: millions of rows spanning 2019 to 2026, one row per ride. Learners write their first queries directly against this table before anything is joined or modeled.

Topics covered:
- What SQL is, and how relational data is structured
- `SELECT`, `FROM`, `WHERE`
- `ORDER BY`, `LIMIT`, and logical operators
- Filtering by text, numeric, and date fields
- Working with timestamps across a multi-year date range

Activities:
- Explore the raw trips dataset in a SQL sandbox (DuckDB or SQLite, no setup required)
- Filter rides by year, month, station, and rider type
- Get a first look at the 2019–2026 date range and how ride volume is distributed across it

Mini-task: Find the three busiest start stations and the year with the highest total ride count.

---

### Week 2: Joins, Aggregates & Grouping

**Goal:** Combine multiple related tables and summarize information across them.

This week introduces multi-table datasets. Learners practice joining records across tables, handling mismatches, and summarizing results at different levels of granularity.

Topics covered:
- `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`
- `GROUP BY` with `COUNT`, `SUM`, `AVG`
- `HAVING` versus `WHERE`
- Date functions for aligning records across tables

Activities:
- Join across multiple related tables
- Identify records with no match and decide how to handle the gap
- Aggregate results by relevant time periods and categories

Mini-project: Build a summarized report combining data from at least two tables, grouped by time period and category.

---

### Week 3: Subqueries, CTEs & Data Cleaning

**Goal:** Write modular queries and handle the inconsistencies that come with real-world data.

Real datasets are rarely clean. Fields go missing, categories shift over time, and trends can be distorted by one-off events. This week is about handling that directly instead of ignoring it.

Topics covered:
- Subqueries in `SELECT`, `WHERE`, and `FROM`
- Common Table Expressions (CTEs)
- `NULL` handling, `COALESCE`, `CASE WHEN`
- Detecting and flagging anomalous periods in a time series

Mini-project: Identify a meaningful trend in the data, and separate genuine change from data gaps or one-off anomalies.

---

### Week 4: Power BI + SQL Mini-Project

**Goal:** Connect the SQL work to a Power BI dashboard.

This is a lighter week by design — the goal is to close the loop from query to visual, not to introduce heavy new SQL concepts.

Activities:
- Connect Power BI to a SQL source
- Write one or two summary queries (totals by category, trend over time)
- Build a dashboard with:
  - A bar chart of top categories
  - A line chart of the trend over time
  - A slicer for a relevant dimension

Deliverables:
- Power BI file (`.pbix`)
- Dashboard screenshot
- Short written reflection on the build

Estimated time: about eight hours.

---

### Week 5: dbt & Data Warehousing Basics

**Goal:** Move from standalone queries to a modeled, testable data pipeline.

By this point the queries have grown in number and complexity. This week restructures that work using dbt, and introduces the concepts behind cloud data warehousing.

Topics covered:
- What dbt is and why it's used in modern analytics
- dbt concepts: sources, staging models, marts
- DAGs and model lineage
- Introduction to data warehouses and Snowflake

Activities:
- Build staging models for the underlying source tables
- Build an intermediate model joining the staged sources
- Add basic dbt tests (not null, uniqueness) and generate documentation
- Review the resulting DAG

---

### Week 6: Final Capstone

**Goal:** Apply everything learned to a single end-to-end deliverable.

The capstone asks learners to produce what a data team would actually be asked for: a clear, well-modeled analysis with a defensible recommendation.

Capstone build:
- SQL to extract and clean the underlying data
- dbt models to transform and document the pipeline
- Power BI dashboard visualizing the key trends

Deliverables:
- Clean SQL scripts
- dbt project with DAG screenshot
- Power BI dashboard
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

A working, end-to-end pipeline: raw SQL queries, a modeled dbt project, and a Power BI dashboard, built progressively over six weeks.