# Overview: A New Dataset, One Last SQL Push, and a Real Dashboard

## Where We're Picking This Up

Weeks 1–3 built real skill against Capital Bikeshare — filtering, joining, aggregating, window functions, and eventually a tested dbt project. That skill is what Week 4 spends, not what it re-teaches. Instead of dashboarding on top of the bikeshare marts, we bring in a new dataset — Olist's Brazilian E-Commerce data on Kaggle — specifically because it isn't flat. Orders, order items, customers, products, sellers, payments, and reviews live in separate tables, which means the SQL from Weeks 1–3 has to be exercised again, for real, before Power BI ever opens.

That's a deliberate trade-off: we lose the head start the bikeshare marts would have given us, but we gain a genuine test of whether the SQL skills transfer to a dataset nobody has seen queries against before.

## Why This Week, and Why Now

- **Monday (Load, join, and shape the data)** exists because Olist arrives as nine loosely related CSVs, not a dashboard-ready table — the exact gap Week 1's joins and Week 2–3's aggregation and window-function work exist to close. This day is a compressed rerun of "raw tables to a reusable summary," proving that skill on new data.
- **Tuesday–Wednesday (Building the dashboard)** exist because a correct SQL result and a usable dashboard are different deliverables — the same distinction Week 3 drew between a correct query and an efficient, organized one.
- **Thursday (Polish, export, reflection)** exists because a "done" dashboard and a "presentable" one aren't the same thing, and Friday's audience won't have context we haven't built into the dashboard itself.
- **Friday (Snowflake, then presenting)** exists because everything this course has run so far — DuckDB, dbt, Power BI — has lived on one machine, and Snowflake is the first look at what changes once a project needs more than that. The presentation closes the week by asking us to explain, out loud, decisions we made silently while building.

So the concrete problem for the week is:

> "We're handed a new, genuinely relational dataset with no prior SQL written against it — can we get from raw CSVs to a dashboard someone else can actually use, using nothing but what we already know?"

## Weekly Breakdown

| Day | Theme | Focus |
|-----|-------|-------|
| Mon | Load, Join & Shape the Olist Data | Load the Olist CSVs into DuckDB, explore the 8–9 table schema, and write the JOINs and aggregations that turn it into one or two clean, Power BI–ready summary tables. |
| Tue | Building the Dashboard — Part 1 | Connect Power BI to the summary tables; bar and line charts for revenue over time and sales by category/state. |
| Wed | Building the Dashboard — Part 2 | Slicers, cross-filtering, and DAX measures layered on top of delivery-time and review-score analysis. |
| Thu | Polish, Export & Reflection | Formatting, titles, a consistent theme, exporting the `.pbix` and screenshot, and writing the short build reflection. |
| Fri | Intro to Snowflake + Presentations | Morning: watch tutorials on Snowflake's architecture and the separation of storage and compute — no hands-on work. Afternoon: present the Power BI dashboard. |

Below is a closer look at what each day actually involves.

**Monday — Load, Join & Shape the Olist Data**
The day starts with getting Olist's `orders`, `order_items`, `customers`, `products`, `sellers`, `payments`, `reviews`, and `geolocation` tables into DuckDB and just looking at them — row counts, key columns, how the translated product-category names and occasional nulls behave. From there it's Week 1's `JOIN`s and Week 2's `CASE WHEN`/`COALESCE` cleanup, chained together to build one or two flattened summary tables: something like an order-level table with category, state, revenue, delivery time, and review score all sitting in one place. This table — not the raw nine — is what Power BI connects to on Tuesday. The judgment call worth naming out loud: how much shaping happens in SQL versus how much gets left for DAX later in the week.

**Tuesday — Building the Dashboard, Part 1**
With a clean summary table in hand, we build the first real visuals: a line chart of revenue over time, and bar charts breaking sales down by product category and by customer state. Same principle as always — line for change over time, bar for comparing categories — but now on data we joined and cleaned ourselves this week instead of inheriting pre-built marts.

**Wednesday — Building the Dashboard, Part 2**
This is where the dashboard becomes interactive. Slicers get added for date range, category, and state, with cross-filtering wired between charts. We also bring in the parts of Olist that reward a second visual angle — delivery time and review score — and write a small number of DAX measures for anything that genuinely needs to happen at render time rather than back in Monday's SQL. That boundary — SQL owns the transformation, DAX owns the display — is the same one drawn in Week 3 between application logic and the dbt layer, just applied to a new tool.

**Thursday — Polish, Export & Reflection**
The last build day is about finishing, not adding. A consistent color theme, readable titles and labels, and a layout that reads top-to-bottom without explanation. We export the `.pbix` file, take the dashboard screenshot due as a deliverable, and write the short reflection — what Monday's SQL work made easier, what surprised us about a dataset built from scratch this week rather than inherited from a pipeline, and what we'd change if we started over.

**Friday — Intro to Snowflake + Presentations**
The morning is deliberately hands-off: tutorials and reading on what a cloud data warehouse is, why teams move off a single machine, and Snowflake's core architectural idea — storage and compute scaled and billed separately, rather than bundled together the way DuckDB has run on our laptop all course. No lab, no account setup — the goal is a mental model, not hands-on knowledge. The afternoon shifts back to what we built this week: each dashboard gets presented, walking through the joins and cleanup behind it, the chart choices, and Thursday's reflection.

## Learning Objectives

By the end of this week, learners will be able to:

- Load a new, multi-table Kaggle dataset into DuckDB and reason about an unfamiliar schema before writing any SQL against it
- Apply Week 1–3 SQL skills — `JOIN`s, `CASE WHEN`/`COALESCE`, aggregation — to flatten a genuinely relational dataset into a small number of dashboard-ready summary tables
- Connect Power BI to a SQL source and import specific prepared tables rather than raw ones
- Choose appropriate chart types (bar vs. line) for a given question, and build slicers and cross-filtering to make a dashboard explorable rather than static
- Write basic DAX measures, and draw a clear line between transformation logic that belongs in SQL and display logic that belongs in DAX
- Export a finished Power BI file and communicate a build process in a short written reflection
- Explain, at a conceptual level, what a cloud data warehouse is and why teams adopt one
- Describe Snowflake's separation of storage and compute, and why that separation distinguishes it from a single-machine setup like DuckDB
- Present a data project to an audience, connecting the upstream SQL work to the dashboard built on top of it

## Difficulty Snapshot — Week 4

Where Week 3 was about efficiency and structure inside a database we already knew well, Week 4 is about applying that same judgment to a dataset seen for the first time, then presenting what came out of it.

| Topic Area | Difficulty | What to Expect |
|------------|------------|-----------------|
| Loading & exploring Olist's schema | Medium | Nothing here is conceptually new, but a genuinely unfamiliar nine-table schema takes real time to get oriented in — budget for it rather than rushing to write joins immediately. |
| Joining & flattening into summary tables | Medium–Hard | This is Week 1–3's skill under real conditions: no answer key, a schema nobody wrote for us, and translated/missing fields that need judgment calls, not just `COALESCE`. |
| Connecting Power BI to DuckDB | Easy–Medium | The connection itself is a few clicks; the more common snag is pointing Power BI at a raw table instead of Monday's cleaned summary table. |
| Bar and line charts | Easy | The visuals themselves are simple to place; the judgment call is matching chart type to the question being asked. |
| Slicers & cross-filtering | Medium | Mechanically easy to add, but designing slicers that make the dashboard more useful rather than more cluttered takes iteration. |
| Basic DAX measures | Medium–Hard | The syntax is unfamiliar, and the real trap is redoing Monday's SQL logic in DAX instead of asking whether DAX is even the right place for it. |
| Dashboard polish & export | Easy | Mostly about restraint — a clean, readable layout beats a feature-packed one. |
| Snowflake architecture (conceptual) | Easy–Medium | No syntax to trip over since it's a watch-and-read day, but the storage/compute separation is a genuinely new mental model worth sitting with rather than skimming. |
| Presenting the project | Medium | Building the dashboard and explaining it clearly to someone who didn't build it are different skills — this is deliberately the first day this course asks for the second one. |

Tip: This week's easiest trap isn't a broken chart — it's rushing Monday to get to the "real" Power BI work sooner. The dashboard is only as trustworthy as the joins underneath it, and Olist's messiness (translated categories, occasional nulls, mismatched keys) is exactly the kind of thing Week 2 already taught us to catch rather than ignore.

## Tools Used This Week

- **Power BI Desktop** — new this week, connected to a fresh **DuckDB** database built specifically for the Olist dataset.
- **Olist Brazilian E-Commerce dataset (Kaggle)** — `orders`, `order_items`, `customers`, `products`, `sellers`, `payments`, `reviews`, and `geolocation` tables, joined and flattened into one or two Power BI–ready summary tables on Monday.
- **DAX** — introduced only for the small amount of display-layer logic Power BI needs that isn't already handled upstream in SQL.
- **Snowflake** (observation only) — no account setup this week; tutorials and documentation only.
- **Git / GitHub repo from Weeks 1–3** — extended with a `week4-olist/` folder for the SQL used to build the summary tables, the `.pbix` file, the dashboard screenshot, and the written reflection.

## End-of-Week Outcomes

By the end of the week, learners should be able to:

- Point to one or two SQL-built summary tables that flatten Olist's nine raw tables into something Power BI can use directly
- Point to a working Power BI dashboard built on those summary tables, with at least one bar chart, one line chart, and working slicers
- Explain, for any visual on the dashboard, which join or aggregation upstream it depends on
- Hand off a `.pbix` file, a dashboard screenshot, and a short written reflection as a complete, presentable deliverable
- Describe in plain language what problem a cloud data warehouse solves, and specifically what Snowflake's separation of storage and compute means in practice
- Present the dashboard to an audience, connecting the visual layer back to the SQL work earlier in the week

This week's deliverable isn't new SQL syntax the way Weeks 1–3 were — it's proof that the SQL already learned transfers to a dataset built from scratch this week, plus a first look at where this kind of project goes once it outgrows a single laptop.

Reflection prompt: Compare building this week's summary tables to Week 1's first joined, aggregated table. Both start from raw, related tables and end with one reusable result — what's different now that it's a dataset seen for the first time, with no course-provided schema to lean on? And where did Monday's SQL work make Tuesday–Thursday's Power BI build easier, versus where did Power BI still demand judgment calls SQL alone couldn't make for us?