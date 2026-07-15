# Overview: Why We're Starting With SQL

## Where We're Picking This Up

Over the last three weeks, we explored the full Capital Bikeshare dataset directly in Power BI — pulling in the raw trip files, building visuals, and getting a first feel for what seven years of ridership data (2019–2026) actually looks like. That exploration was useful, but it also surfaced a real problem: loading all seven years of trip data into Power BI at once is slow. Refreshes take a long time, the file gets heavy, and every small change to a visual means waiting on the model to recalculate against millions of raw rows.

That slowdown isn't a Power BI mistake — it's a sign of a missing layer. Power BI is built to visualize data efficiently, not to filter, clean, and aggregate seven years of raw transactional records efficiently. That work belongs upstream, in a proper queryable table, so that by the time the data reaches Power BI, it's already been narrowed down to what's actually needed. This week is about building that upstream layer using SQL.

## Why SQL, and Why Now

This is the point where the course shifts from an analytics tool (Power BI) to a data engineering foundation (SQL). The difference matters:

- A CSV loaded into Power BI has to be scanned and processed in full, every time, inside the visualization tool itself.
- A well-structured SQL table lets us filter, aggregate, and shape the data *before* it ever reaches a dashboard — so Power BI only ever touches the smaller, already-summarized result.

This is a core data engineering idea, not just a SQL syntax lesson: push the heavy lifting as far upstream as possible, and only hand downstream tools (like Power BI) the data they actually need. Week 1 is the first step toward that — getting comfortable enough with SQL to query the seven-year trip dataset directly, instead of relying on Power BI to do that work for us.

So we have a concrete problem to work through this week:

> "We have seven years of trip data, and loading all of it into Power BI grinds to a halt. Before we touch a dashboard again, can we understand the data well enough in SQL to know what to aggregate, what to filter out, and what a leaner, query-ready table should actually contain?"

The job this week is to investigate, not memorize. We explore the raw trip data using Beekeeper Studio, ask direct questions of it, and build a first real insight: a station-level ride report built entirely from a SQL query — no Power BI refresh required to get there. Each day adds one new layer of SQL, and each layer also introduces a data engineering habit that becomes more important as the dataset and the pipeline grow in later weeks.

Each day is broken down into modular lessons — individual markdown files that focus on one concept at a time, with a worked example against the trip data and a short exercise to try independently.

## Weekly Breakdown

| Day | Theme | Discovery Focus |
|-----|-------|------------------|
| 1 | What's in Our Trip Data? | Understand the structure of the trips table: start/end stations, timestamps, rider type, duration, bike type. What does each column represent, and why does structure in SQL matter more than structure in a spreadsheet? |
| 2 | Which Stations Matter Most? | Filter and sort trip records to find the busiest start and end stations, and see how much faster this is against a table than scanning raw CSVs in a BI tool. |
| 3 | Rides by Year and Season | `GROUP BY` year and month to see how ridership has moved across seven years — and start thinking about which of these summaries should become a reusable table instead of a one-off query. |
| 4 | Trends Over Time | Slice the data by date at a finer grain, and pull together a lean, pre-aggregated result that could be handed to Power BI without dragging in all seven years of raw rows. |

Below is a closer look at what each day actually involves.

**Day 1 — What's in Our Trip Data?**
Before writing a single meaningful query, the first task is to understand what's actually in the table — column names, data types, and how consistent the fields are across all seven years. This is also where the data engineering mindset starts: in Power BI, a messy column can often be patched over visually. In SQL, we deal with it at the source, once, so every query and every downstream tool benefits from the fix instead of just one report.

**Day 2 — Which Stations Matter Most?**
With the structure understood, this day introduces filtering and sorting together — narrowing to a specific year or station type, then ranking the results. It's also the first place the performance difference becomes obvious: a filtered query against a table returns in a fraction of the time it takes Power BI to filter the same thing after loading everything into memory first.

**Day 3 — Rides by Year and Season**
This is where `GROUP BY` is introduced properly, summarizing across the full seven-year window instead of looking at individual trips. It's also where we introduce a genuine data engineering question: instead of running this aggregation fresh every time, should the result be saved as its own summary table? That question — raw table versus pre-aggregated table — is a preview of what staging and mart layers will do later in the course with dbt.

**Day 4 — Trends Over Time**
The final day narrows the time grain further, comparing weekday and weekend ridership and tracking how that split has changed over the years. It closes by producing a small, clean, query-ready result set — the kind of table that, if connected to Power BI instead of the raw seven-year file, would load in seconds rather than minutes.

Each day lives in its own folder (`Day-1`, `Day-2`, and so on) and contains multiple `.md` files — one per concept — so the material can be worked through step by step, with room to pause and revisit a concept before moving on.

## Learning Objectives

By the end of this week, learners will be able to:

- Read and explore a structured SQL dataset (`trips`) spanning multiple years, including recognizing column types and spotting inconsistencies before they cause query errors
- Load and query CSV files using Beekeeper Studio, without needing a database server set up first
- Write basic SQL queries: `SELECT`, `WHERE`, `ORDER BY`, and combine them into a single readable query
- Filter records by date range across a seven-year span, including isolating a single year, a single month, or a custom range
- Use `GROUP BY` alongside `COUNT`, `SUM`, and `AVG` to summarize ride volume and duration at the station, month, and year level
- Explain, in plain terms, why filtering and aggregating in SQL is faster and more scalable than loading raw data directly into a BI tool
- Recognize the early signs of when a query result should become its own reusable table rather than being recalculated every time — the first step toward staging tables and pipelines
- Build a basic station-wise and year-wise ride report suitable for handing to a non-technical stakeholder, with numbers that can be defended if questioned

## Difficulty Snapshot — Week 1

The ramp this week goes from data exploration to first insights, building comfort with querying before anything gets complicated. Nothing this week requires a join or a subquery — the goal is to be genuinely comfortable with the basics before Week 2 introduces multiple tables.

| Topic Area | Difficulty | What to Expect |
|------------|------------|-----------------|
| SELECT basics | Easy | Reading trip data is intuitive — close to a spreadsheet view. The main skill here is knowing which columns to look at first. |
| WHERE filters | Easy | Simple conditions narrow the dataset down quickly, including by year, station, or rider type. Combining two or three conditions is where it gets slightly less obvious. |
| Date filtering | Medium | Seven years of timestamps means learning to filter and format dates carefully — extracting just the year or month from a full timestamp is a common early stumbling block. |
| GROUP BY & SUM/COUNT | Medium | This is the first real shift in thinking: from looking at individual rows to summarizing across thousands of them at once. It usually takes a few attempts to get the grouping level right. |
| Station-level sorting | Medium | Ranking stations by volume takes a bit of trial and error, particularly once ties or near-ties in the data start showing up in the results. |
| Query vs. table thinking | Medium | Deciding whether a result should stay as a one-off query or be saved as its own table is a new kind of judgment call — less about syntax, more about how the data will actually be used downstream. |

Tip: With seven years of data, it's easy to run into unexpected results — a station that didn't exist in 2019, a gap in one year's records, a count that looks too high or too low because of a formatting issue in the timestamp column. When that happens, the instinct should be to pause, check the query piece by piece, and rethink the assumption rather than assume the data is simply wrong. That habit — questioning the query before questioning the data — is a large part of how real analysts and data engineers build trust in their own numbers.

## Tools Used This Week

- **DuckDB** — a lightweight SQL engine built specifically for querying CSV and columnar files directly, without needing to load them into a full database server first. It's fast even on a dataset with several years of trip records, and it runs locally through Python or the command line — a useful contrast to how slow the same data felt inside Power BI.
- **CSV dataset:** `trips.csv` — Capital Bikeshare trip records covering 2019 through 2026. Each row represents a single ride, with fields for start and end station, start and end time, rider type, and bike type.
- **Beekeeper Studio** — a free SQL client with a clean interface for writing queries, browsing table structure, and viewing results in a spreadsheet-like grid. It connects directly to the local DuckDB file, so no server setup or credentials are required.
- **Code editor or notebook** — VS Code, Jupyter, or Streamlit, any of which work fine for following along; none are required, but having one open makes it easier to save queries as they're written rather than losing them at the end of a session.

Beekeeper Studio makes SQL feel fast, clean, and local — a good fit for first-time analysts, and a good habit to build before moving to a hosted warehouse like Snowflake later in the course, where the same query logic applies but the setup is heavier.

## End-of-Week Outcomes

By the end of the week, learners should be able to:

- Query raw trip data confidently using `SELECT`, `WHERE`, and `ORDER BY`, chaining conditions together without needing to look up syntax each time
- Filter and compare data across specific years within the 2019–2026 range, and explain what changed between any two years chosen
- Use `GROUP BY` to find top stations, busiest months, or highest-volume years, and know which aggregate function fits which question
- Explain, using this week's experience as the example, why the Power BI slowdown happened and what a SQL-first approach changes about that
- Present a short summary to the team: which stations lead, how ridership has changed since 2019, and one recommendation for where to look next

This summary doesn't need to be long or polished. What matters is that every number in it can be traced back to a specific query, and that the person presenting it can explain, on the spot, why they grouped the data the way they did — and why that query would now load into Power BI far faster than the raw seven-year file did.

Reflection prompt: Compare this week's SQL exploration to the earlier Power BI-only exploration. What could you find out about the seven-year trip data now that felt harder to get to before? Where do you still think Power BI is the right tool, and where does the work clearly belong in SQL instead?