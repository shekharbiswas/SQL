# Getting Started with Your Database

Before running any SQL, it's worth getting clear on something basic:

> What is a database? And what exactly is a table?

If you've used spreadsheets like Excel or Google Sheets before, you already understand more of this than you might think.

## What Is a Database?

A database is a structured way to store information on a computer. Instead of one large file, it organizes data into separate tables — similar to separate sheets inside one Excel workbook, or separate folders on a computer.

For this week, we'll use a format called SQLite (or DuckDB, which works similarly for our purposes), which stores everything in a single `.db` file — something like `bikeshare_2019_2026.db`.

You'll open that file using Beekeeper Studio, the SQL editor where you write queries and view results.

## What Is a Table?

A table is a collection of similar data — like a single spreadsheet with rows and columns.

Every table has:

- **Columns** — the types of information stored (like a station name, a timestamp, or a rider type)
- **Rows** — each individual record (in our case, one bike ride)

## The Table You'll Be Working With

Once connected to your database, you'll see one main table: `trips`.

This table holds one row per Capital Bikeshare ride, across all seven years, 2019 through 2026.

Typical columns include:

- `start_time` — when the ride began
- `end_time` — when the ride ended
- `start_station` — where the ride started
- `end_station` — where the ride ended
- `rider_type` — member or casual
- `bike_type` — classic or electric, depending on what the data includes for that period

A single row might represent one ride that started at a station near a metro stop on a Tuesday morning and ended a few kilometers away twenty minutes later. Multiply that by seven years and every station in the system, and you get a sense of the scale we're working with — millions of rows, all following the same simple structure.

## What You'll Do Next

Right now, this table doesn't exist yet as an actual database file — it's still sitting as raw CSV files, the way Capital Bikeshare publishes its data. The next lesson walks through building it: combining seven years of monthly CSVs into a single `trips` table inside your own database file, which you'll then open in Beekeeper Studio.

From there, working with the data is just a matter of asking it questions — using SQL instead of filters or pivot tables.