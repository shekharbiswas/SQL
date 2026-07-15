# Welcome to Week 1: The Capital Bikeshare Trip Data

Welcome to the first week of hands-on SQL. You've already spent three weeks inside Power BI, working with the Capital Bikeshare dataset directly — seven years of ride records, 2019 through 2026. This week, the goal is different: instead of visualizing the data, we're going to understand it well enough in SQL to make everything that comes after it — including Power BI — faster and easier to work with.

Capital Bikeshare is a real, large bike-share system, and the dataset behind it isn't small or clean by accident. It's grown, changed, and gone through a genuinely disruptive year. That's exactly the kind of dataset worth learning SQL on, instead of a toy example that behaves the way a textbook wants it to.

## Your Focus This Week

You won't be given a fixed checklist of queries to run. You'll get a real dataset, a few concrete questions, and the job of figuring out how to get from one to the other. That's deliberate — this is close to how the work actually happens once you're past the tutorial stage.

Over the week, you'll effectively sit in the position of the people who'd normally ask for this data, and think about what they'd actually need:

| Who's Asking | What They Want to Know | Why It Matters |
|---------------|--------------------------|------------------|
| Operations / rebalancing teams | Which stations run out of bikes or fill up fastest | Determines where bikes need to be moved, and when |
| System planning | Which stations have grown or declined in usage over the years | Informs where new stations get added or existing ones get resized |
| Reporting / leadership | How ridership has trended since 2019, in a way that's fast to refresh | The exact question that's been slow to answer in Power BI until now |
| You, this week | All of the above, but from raw SQL first | Your job is to turn slow, heavy raw data into something quick to query and easy to hand off |

You won't formally rotate through these roles like separate jobs. But keeping their questions in mind is what turns a SQL exercise into something that actually resembles data engineering work.

## The Data You'll Use

This week centers on one file:

`trips.csv` — Capital Bikeshare trip records, 2019 through 2026. Each row is a single ride.

Fields typically include:
- `trip_id`
- `start_station`
- `end_station`
- `start_time`
- `end_time`
- `rider_type` (member or casual)
- `bike_type`

Your task this week is to get comfortable enough with this table to answer a genuinely useful question:

> Based on seven years of ride data, which stations and time periods should we be paying the most attention to — and why has it taken this long to find that out through Power BI alone?

## Tools You'll Use

You'll write and run SQL queries using Beekeeper Studio — a clean, beginner-friendly SQL editor that works well with local databases like DuckDB or SQLite.

Download it here: [www.beekeeperstudio.io](https://www.beekeeperstudio.io)

Your workflow this week:
1. The `trips.csv` file gets loaded into a local database (DuckDB or SQLite).
2. You open that database file in Beekeeper Studio.
3. From there, you write SQL queries directly against the trips table — no waiting on a Power BI model to refresh in the background.

Why Beekeeper Studio:
- Easy setup and a clean interface
- Works offline, no registration required
- Lets you browse tables and results visually, which helps while you're still building intuition for the data
- Close to the tools real analysts and data engineers use day to day

This workflow is also the point: querying a local database directly is noticeably faster than reloading a multi-year CSV into a BI tool every time a question changes. You'll feel that difference by the end of the week, not just read about it.

## What Success Looks Like This Week

You're not expected to get everything right on the first attempt. You're expected to stay curious and keep testing what the data is actually telling you.

By the end of the week, you should be able to:
- Run your first `SELECT` and `WHERE` queries against the trips table
- Explore the shape of the data — what's missing, what's inconsistent across the seven-year span
- Start thinking in metrics: ride count versus average duration versus weekday/weekend split
- Use `GROUP BY` to summarize by station, month, and year
- Produce a first real result: a short, defensible answer to which stations and periods matter most

By the end, you'll be ready to present:
- Top stations by ride volume
- A short insight: where should attention (rebalancing, planning, or reporting) focus next, and why

## Keep This in Mind

- You're here to explore, not to memorize syntax.
- Getting stuck is part of the process. Debugging a query is a bigger part of the job than writing one correctly on the first try.
- There often isn't one right answer — only a better-supported one.
- Keep asking: would this actually help someone make a decision, or is it just a number that looks interesting?

Journal prompt: Now that you've spent three weeks in Power BI and are starting to work directly in SQL, what feels different? What could you check about the seven-year trip data this week that felt out of reach before?

You're ready to start. Begin by loading the data and looking at the first hundred rows.