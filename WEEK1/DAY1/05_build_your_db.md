# Build Your Own Bikeshare Database

Unlike a typical course dataset, there's no prebuilt `.db` file waiting for you here. Capital Bikeshare publishes its trip data as raw CSV files, one per month, going back years. Turning that into a single queryable database is itself the first real data engineering task of the course — and a useful one, since this exact step is what most real analytics work starts with.

This lesson walks through doing that using DuckDB, the same lightweight SQL engine mentioned earlier in the week. DuckDB can read CSV files directly and write the result into a proper database file that Beekeeper Studio can then connect to.

## Step 1: Get the Raw Data

Capital Bikeshare's trip history is published publicly, broken into monthly CSV files, at:
`https://s3.amazonaws.com/capitalbikeshare-data/index.html`

For this course, download the files covering 2019 through 2026 and place them all in one folder, for example:

```
data/
  2019-01.csv
  2019-02.csv
  ...
  2026-06.csv
```

Worth knowing before you go further: Capital Bikeshare changed its file format partway through this range. Trip files from roughly 2019 through early 2020 use one set of column names, and files from around mid-2020 onward use a different set — a change that came from a broader shift across Lyft-operated bike-share systems to a standardized trip-data format. The two schemas describe the same underlying information, but the column names, capitalization, and even units differ. That mismatch is exactly why we can't just point DuckDB at every CSV in the folder and expect a clean, aligned table — it needs to be handled explicitly, which is what the rest of this lesson walks through.

The older files (roughly 2019–early 2020) typically look like this:

| Column | Meaning |
|--------|---------|
| `Duration` | Ride length, in seconds |
| `Start date` | Ride start timestamp |
| `End date` | Ride end timestamp |
| `Start station` | Start station name |
| `End station` | End station name |
| `Bike number` | Identifier for the bike used |
| `Member type` | `Member` or `Casual` |

The newer files (roughly mid-2020 onward) typically look like this:

| Column | Meaning |
|--------|---------|
| `ride_id` | Unique ride identifier |
| `rideable_type` | Bike type (classic, electric, etc.) |
| `started_at` | Ride start timestamp |
| `ended_at` | Ride end timestamp |
| `start_station_name` | Start station name |
| `end_station_name` | End station name |
| `member_casual` | `member` or `casual` |

Take a moment to actually open one file from 2019 and one file from 2022 or later side by side before moving on. Seeing the mismatch firsthand makes the next step — standardizing both formats into one table — a lot more intuitive than just being told about it.

## Step 2: Install DuckDB (Command-Line Version)

**Important:** `pip install duckdb` installs the *Python library*, not the standalone command-line tool. If you run `pip install duckdb` and then try `duckdb bikeshare.db` in your terminal, you'll get `'duckdb' is not recognized as an internal or external command` — the CLI binary is a separate download.

To get the actual command-line tool on Windows:

1. Go to [duckdb.org/docs/installation](https://duckdb.org/docs/installation/)
2. Select **CLI**, **Windows**, **x86_64**
3. Download the zip — it contains a single file, `duckdb.exe`
4. Extract `duckdb.exe` into the folder where your CSV files (and your future `.db` file) will live, e.g. `C:\Users\shekh\Downloads\`

From then on, run it directly from that folder:

```bash
cd C:\Users\shekh\Downloads
duckdb.exe bikeshare_2019_2026.db
```

This drops you into the DuckDB shell, ready for the SQL steps below.

*(Optional: to run `duckdb` from any folder without `cd`-ing in first, add the folder containing `duckdb.exe` to your Windows PATH via System Properties → Environment Variables → Path → New.)*

## Step 3: Standardize Each Schema Era Separately

Because the older and newer files use genuinely different column names — not just a reordering — a single `read_csv_auto('data/*.csv')` call would either fail outright or, worse, silently produce a table full of mismatched or empty columns. So instead of combining everything in one step, we standardize each era to a common set of column names first, and only then bring them together.

From inside the DuckDB shell opened in Step 2, first load the older files (2019 through early 2020) into a staging table, renaming columns to a common naming scheme as we go:

```sql
CREATE TABLE trips_legacy AS
SELECT
    "Bike number"        AS bike_id,
    "Start date"          AS start_time,
    "End date"            AS end_time,
    "Start station"       AS start_station,
    "End station"         AS end_station,
    "Member type"         AS rider_type,
    CAST(NULL AS VARCHAR) AS bike_type
FROM read_csv_auto('data/2019-*.csv', union_by_name = true);
```

Then load the newer files (mid-2020 onward) into a second staging table, using their column names:

```sql
CREATE TABLE trips_modern AS
SELECT
    ride_id             AS bike_id,
    started_at           AS start_time,
    ended_at             AS end_time,
    start_station_name   AS start_station,
    end_station_name     AS end_station,
    member_casual         AS rider_type,
    rideable_type          AS bike_type
FROM read_csv_auto('data/2020-06*.csv', 'data/2021-*.csv', 'data/2022-*.csv',
                    'data/2023-*.csv', 'data/2024-*.csv', 'data/2025-*.csv',
                    'data/2026-*.csv', union_by_name = true);
```

Adjust the file patterns to match exactly where the format actually changes in your downloaded files — check a couple of files around early-to-mid 2020 directly, since Capital Bikeshare's own transition date is the one detail worth confirming rather than assuming.

Two more details worth noticing in these queries:

- `rider_type` ends up holding `Member`/`Casual` from the old files and `member`/`casual` from the new ones — same information, different capitalization. That inconsistency is worth flagging now; it's exactly the kind of thing that causes a `GROUP BY rider_type` later in the week to quietly return four categories instead of two.
- `bike_type` doesn't exist in the older files at all, so it's filled with `NULL` in `trips_legacy`. That's an honest gap in the source data, not a mistake in the query — and it's worth remembering when you get to any analysis involving bike type, since it simply won't be answerable for 2019 and early 2020.

## Step 4: Combine Both Eras into One Table

With both staging tables using the same column names, they can now be safely combined:

```sql
CREATE TABLE trips AS
SELECT * FROM trips_legacy
UNION ALL
SELECT * FROM trips_modern;
```

This gives a single `trips` table spanning the full 2019–2026 range, with a consistent set of columns throughout — even though the two halves of the data started out looking quite different.

## Step 5: Do a Quick Sanity Check

Before moving on, confirm the load actually worked as expected — and specifically check for the two issues introduced by the schema change:

```sql
SELECT COUNT(*) FROM trips;
```

```sql
SELECT MIN(start_time), MAX(start_time) FROM trips;
```

```sql
SELECT DISTINCT rider_type FROM trips;
```

The row count should be in the millions, and the date range should span from somewhere in 2019 to somewhere in 2026. The `rider_type` check is the important one this time: if it returns more than two distinct values (for example, `Member`, `Casual`, `member`, and `casual` all showing up separately), that's the capitalization mismatch from Step 3 surfacing exactly where expected — worth cleaning up now, with a simple `LOWER(rider_type)`, rather than discovering it mid-query later in the week.

When you're done, exit the DuckDB shell:

```sql
.quit
```

## Step 6: Connect Beekeeper Studio to the Database

With `bikeshare_2019_2026.db` created, go back to Beekeeper Studio:

1. Click "New Connection"
2. In **Connection Type**, select **DuckDB** (not SQLite — DuckDB files aren't SQLite files, and Beekeeper will reject them with a "file is not a database" error if the wrong type is selected)
3. Click "Choose a Database File"
4. Select `bikeshare_2019_2026.db`
5. Click Connect

The `trips` table should now appear in the left sidebar, ready to query.

*Note: DuckDB support depends on your Beekeeper Studio version — if "DuckDB" doesn't appear in the Connection Type dropdown, update Beekeeper Studio to a version that includes it.*

## Why This Step Matters

It would have been faster to hand you a finished `.db` file and skip straight to writing `SELECT` statements. But building the database yourself, from the same raw files Capital Bikeshare actually publishes, is a more honest starting point — real data doesn't arrive pre-cleaned into a single table, and knowing how to get it there is as much a part of this course as writing the queries afterward.

From here, the rest of Day 1 assumes this `trips` table exists and is ready to explore.
