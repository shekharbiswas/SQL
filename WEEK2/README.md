# Overview: From Single Queries to Trustworthy Ones

## Where We're Picking This Up

Week 1 got us comfortable with the basics: `SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`, and enough date filtering to build a first station-level ride report. That was enough to answer straightforward questions about the seven-year trip dataset — but it also left a trail of loose ends. Some questions ("which stations are above their neighborhood's average?") didn't have a clean single-query answer yet. Some data problems — the unmatched-station gap, in particular — got noted and worked around rather than actually fixed. And every query so far has stood alone, with no way to check whether a number from Monday still holds up on Friday.

Week 2 is about closing those loops. We move from queries that answer one question in isolation to queries that build on each other, handle messy real-world data deliberately instead of by accident, and end with a genuine investigative finding rather than just a number.

## Why This Week, and Why Now

This week has a clear arc, and each day exists because the day before it created a need:

- **Subqueries (Day 1)** let a query answer a question that needs an intermediate result first — "stations above average," for instance — without yet worrying about readability.
- **CTEs (Day 2)** exist because some of Day 1's subqueries, while correct, were hard to trust at a glance. Naming each stage of a calculation makes a multi-step query something a teammate could actually read.
- **NULL handling (Day 3)** exists because the CTE habit from Day 2 makes it possible to finally go back and *fix* the unmatched-station problem from Week 1, instead of just filtering around it every time.
- **Anomaly detection (Day 4)** exists because once the data is cleaner, we can trust a time series enough to ask a bigger question: is ridership actually changing, or is what looks like a trend really a data gap or a one-off event?
- **Consolidation (Day 5)** exists because two weeks of query history, notes, and fixes don't organize themselves — and a repo that isn't reviewed regularly stops being trustworthy no matter how good the individual queries were.

So the concrete problem for the week is:

> "We can answer one-off questions in SQL, but we can't yet build on our own results, trust our own data quality, or tell a real trend apart from a data artifact. Can we get to a point where a finding in this dataset is actually defensible?"

## Weekly Breakdown

| Day | Theme | Discovery Focus |
|-----|-------|------------------|
| 1 | Subqueries | Answer questions that need an intermediate result first — scalar subqueries in `WHERE`, subqueries in `FROM`, and correlated subqueries — and feel where they start to get hard to read. |
| 2 | Common Table Expressions | Rewrite the hardest Day 1 subqueries as named, readable `WITH` clauses, then go further: multiple CTEs and chained, multi-stage CTEs. |
| 3 | NULL Handling, COALESCE & CASE WHEN | Go back and actually resolve the Week 1 unmatched-station gap, using `COALESCE` for defaults and `CASE WHEN` for categorization and conditional counts. |
| 4 | Detecting Anomalies (Mini-Project) | Turn `trips` into a real daily time series, find missing periods, compare periods against each other, and flag statistical outliers — then investigate a few until they're explained, not just listed. |
| 5 | Consolidating the Repo | Organize two weeks of SQL and notes into a coherent, current repo: updated README, cleaned-up file structure, and data-quality notes that reflect what's actually true now. |

Below is a closer look at what each day actually involves.

**Day 1 — Subqueries**
The day starts from real Week 1 questions that a single flat query couldn't answer cleanly — like finding stations above average capacity — and introduces the three main subquery shapes: scalar subqueries dropped into `WHERE`, subqueries used as a virtual table in `FROM`, and correlated subqueries that re-run for every row of the outer query. It's also where the classic `NOT IN` plus `NULL` pitfall shows up for the first time, planting a seed that Day 3 comes back to deliberately.

**Day 2 — Common Table Expressions**
With subqueries under our belt, this day is about readability rather than new logic. The same questions get rewritten using `WITH`, first as a one-to-one swap, then as multiple independent CTEs joined together, and finally as chained CTEs where each stage builds on the last. The mini-project stitches per-station totals, neighborhood joins, and neighborhood averages into one multi-stage report — the same kind of "raw versus reusable" thinking Week 1 first raised, now actually put into practice.

**Day 3 — NULL Handling, COALESCE & CASE WHEN**
This is the day the unmatched-station problem stops being a footnote. `COALESCE` supplies deliberate defaults instead of letting `NULL`s vanish silently from a `GROUP BY`, and `CASE WHEN` builds real categories — capacity tiers, duration buckets, a `'Matched'`/`'Unmatched'` flag — directly in SQL. By the end of the day, the exact percentage of rides affected by the station gap is known, `stations_summary` is optionally rebuilt on cleaner logic, and the Week 1 data-quality notes get their first real update rather than just an addition.

**Day 4 — Detecting Anomalies & the Week 2 Mini-Project**
The cleaned-up data finally gets treated as a time series: grouped by day, checked against a full calendar to separate real zero-ride days from actual gaps in the data, compared period-over-period, and flagged using z-scores — first globally, then per month once seasonality turns out to matter. The mini-project is the payoff: narrowing down to two or three genuinely interesting anomalies and working through whether each one is a data gap, a weather effect, or a real event, before writing up a short, defensible finding.

**Day 5 — Consolidating Week 2 in the Repo**
The final day doesn't add new SQL concepts — it makes sure two weeks of work actually holds together as one project. Every day's working queries get committed to `sql/`, the README gets a Week 2 section and an honest rewrite of "Known Data Issues," the data-quality notes get updated in place rather than just appended to, and the whole repo gets a fresh, stranger's-eye read-through before Week 3 begins.

## Learning Objectives

By the end of this week, learners will be able to:

- Write scalar, `FROM`-clause, and correlated subqueries, and recognize when each shape fits the question being asked
- Rewrite subqueries as CTEs, and chain multiple CTEs together into a readable, multi-stage transformation
- Judge when a CTE is genuinely earning its complexity versus when a single `GROUP BY` with a `JOIN` would answer the same question more simply
- Explain the difference between `NULL`, an empty string, and a placeholder value, and avoid the `NOT IN`/`NULL` pitfall using `NOT EXISTS` instead
- Use `COALESCE` to supply deliberate defaults, and `NULLIF` to convert placeholder values into real `NULL`s first
- Use `CASE WHEN` to build categorical tiers and conditional counts inside a single `SELECT`, without separate filtered queries
- Build a full daily time series from raw trip data, detect genuine gaps versus real zero-ride days, and compare periods against each other without a window function
- Flag statistical anomalies using z-scores, adjust for seasonality by computing statistics per month rather than globally, and separate a real anomaly from a data artifact
- Keep a growing repo — SQL files, README, and data-quality notes — current and coherent across multiple weeks of work, rather than just additive

## Difficulty Snapshot — Week 2

Where Week 1 was about getting comfortable with single queries, Week 2 is about combining them, trusting the data underneath them, and turning a flagged pattern into an actual explained finding.

| Topic Area | Difficulty | What to Expect |
|------------|------------|-----------------|
| Scalar & FROM subqueries | Medium | Conceptually straightforward once seen once, but nesting them without losing track of which result feeds which outer query takes some getting used to. |
| Correlated subqueries | Hard | The re-run-per-row behavior is the first genuinely non-obvious concept this week; the `NOT IN`/`NULL` pitfall trips people up here specifically. |
| CTEs (single & multiple) | Easy–Medium | Mostly a readability upgrade over subqueries already written; the syntax itself isn't the hard part. |
| Chained CTEs | Medium | Deciding how many stages to break a problem into, and naming each one clearly, is more of a design skill than a syntax one. |
| NULL semantics | Medium | `IS NULL` versus `= NULL` is simple once demonstrated, but predicting *where* `NULL`s will silently affect a `GROUP BY` or a `NOT IN` takes real attention. |
| COALESCE & CASE WHEN | Easy–Medium | The syntax is approachable; the harder judgment call is choosing a sensible default versus deliberately leaving a value as `NULL`. |
| Time series & anomaly detection | Hard | Building the series and flagging outliers is mechanical; the real difficulty is judgment — separating a gap, a seasonal pattern, and a genuine anomaly from each other. |

Tip: This week's hardest moments aren't syntax errors — they're the moments where a query runs fine and returns a plausible-looking wrong answer, like a `NOT IN` silently dropping rows because of a `NULL`, or an anomaly flag firing because of seasonality rather than a real event. The habit worth building this week is the same one Week 1 started: question the query and the assumption behind it before trusting the number.

## Tools Used This Week

- **DuckDB**, queried through **Beekeeper Studio** — the same setup from Week 1, now used for multi-stage CTEs and time-series queries against the full seven-year `trips` table.
- **`stations` table** — joined against `trips` throughout the week, and the source of the unmatched-station gap that Day 3 finally resolves.
- **NOAA weather data** (if loaded) — used in Day 4's mini-project as one possible explanation for a flagged anomaly, alongside data gaps and known external events.
- **Git / GitHub repo from Week 1** — extended rather than replaced this week, with four new SQL files, updated notes, and a rewritten README as of Day 5.

## End-of-Week Outcomes

By the end of the week, learners should be able to:

- Take a question that needs an intermediate result and answer it two ways — as a subquery and as a CTE — and explain which one they'd hand to a teammate and why
- Point to the exact percentage of rides affected by the Week 1 station gap, and explain how `stations_summary` now accounts for it
- Build a clean daily time series from raw trip data and tell the difference between a true data gap and a real zero-ride day
- Name one specific anomaly or trend in the seven-year dataset, and walk through how a data gap, a weather effect, and a known external event were each ruled in or out as the explanation
- Open the GitHub repo as a stranger would and follow a coherent story from Week 1's foundational build through Week 2's advanced querying and cleanup

This week's deliverable isn't a single number the way Week 1's station report was — it's a repo that holds together end to end: working SQL for subqueries, CTEs, NULL handling, and anomaly detection, a data-quality record that's current rather than just additive, and one real, defended finding about the data.

Reflection prompt: Compare this week's investigative finding to Week 1's station-level report. Week 1 asked "what does the data show?" — Week 2 asked "can we trust what the data shows, and why?" Where did that shift in question change how you actually worked, and where do you think it'll matter most once Week 3 introduces window functions and dbt?