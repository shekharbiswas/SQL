# Day 5 Task: Packaging Week 1 Into a GitHub Repo

**Total time: 6 hours**

Four days of work now exist only as query history and a handful of tables sitting in a local database: `trips`, `stations`, and `stations_summary`. None of it is saved anywhere durable, and none of it would make sense to anyone but you if they opened your database right now. Today fixes that. You'll turn Week 1 into an actual GitHub repository — organized, documented, and readable by someone who wasn't in the room for any of it.

This is also, practically, the first entry in your portfolio. Every week from here adds to the same repo.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Git & GitHub setup | 45 min |
| 2 | Structuring the repo | 75 min |
| 3 | Writing the project README | 90 min |
| 4 | Committing Week 1's SQL | 60 min |
| 5 | Documenting data quality notes | 45 min |
| 6 | Final polish & reflection | 45 min |

---

## Block 1: Git & GitHub Setup (45 min)

**Goal:** Get a real repo initialized and connected to GitHub — not a folder of files you'll zip up later.

### Do this: 6 tasks

1. Install Git if you haven't already, and confirm it works with `git --version`.
2. Create a new, empty repository on GitHub — public, so it's visible in your portfolio — named something clear like `bikeshare-sql-project`.
3. On your machine, create a local project folder and run `git init` inside it.
4. Connect the local folder to the GitHub repo with `git remote add origin <your-repo-url>`, and confirm it with `git remote -v`.
5. Create a `.gitignore` file and exclude anything that shouldn't be tracked — your actual database file (`.db` / `.duckdb`), any local config, and raw CSVs if they're large.
6. Make an empty first commit (`git commit --allow-empty -m "Initial commit"`) and push it, just to confirm the connection to GitHub actually works before you add real content.

---

## Block 2: Structuring the Repo (75 min)

**Goal:** Set up a folder structure that will still make sense in Week 6, not just today.

### Do this: 7 tasks

1. Create a `sql/` folder — this is where every query file from the week will live.
2. Inside `sql/`, create one file per day: `day1_setup.sql`, `day2_filtering.sql`, `day3_aggregation.sql`, `day4_joins_summary.sql`.
3. Create a `notes/` folder for anything that isn't SQL — data quality observations, reflections, decisions you made and why.
4. Decide, and write down in one sentence, whether raw data belongs in this repo at all given its size — and if not, where you'll note that it lives instead.
5. Create a top-level `README.md` (empty for now — Block 3 fills it in).
6. Look at the folder structure as a whole and ask: if you added Week 2's files right now using the same pattern, would it stay organized, or would it need to change? Adjust now if it wouldn't.
7. Commit the folder structure itself, even with mostly empty files, so the shape of the project is captured in Git history early.

---

## Block 3: Writing the Project README (90 min)

**Goal:** Write the document that decides whether anyone looking at this repo — including future-you — understands what it is in the first thirty seconds.

### Do this: 8 tasks

1. Write an opening paragraph: what dataset is this, how big is it, and what's the project about. Assume the reader has never heard of Capital Bikeshare.
2. Add a "Data Source" section describing where `trips` and `stations` came from, and the years covered.
3. Add a "Setup" section: what database this was built in, and roughly how someone would recreate it (even if the raw files aren't included).
4. Add a "What's in `sql/`" section briefly describing what each day's file covers — one line per file is enough.
5. Add a short "Key Findings" section highlighting one or two genuinely interesting things you found this week — busiest stations, ridership patterns, whatever stood out.
6. Add a "Known Data Issues" section that references the unmatched-station problem from Day 4, Block 3 — don't hide the gap, name it.
7. Add a "Next Steps" section noting, briefly, that Week 2 builds on this with subqueries, CTEs, and deeper data cleaning.
8. Read the README start to finish as if you'd never seen the project before. Fix anything that assumes context the reader doesn't have.

---

## Block 4: Committing Week 1's SQL (60 min)

**Goal:** Get your actual working queries into the repo, cleaned up enough to be read by someone else.

### Do this: 7 tasks

1. Go back through your Day 2–4 work and copy your final, working queries into the matching files in `sql/` — not every attempt, just the ones that worked.
2. Add a short comment above each query explaining, in one line, what question it answers.
3. Make sure every query uses the aliasing and formatting conventions from Day 4, Block 2 — consistent style across all four files.
4. Include the full `CREATE TABLE stations_summary AS SELECT ...` statement from Day 4 in `day4_joins_summary.sql`, exactly as it was used to build the deliverable table.
5. Run each file's queries one more time against your database to confirm they still work exactly as written, with nothing missing that only existed in your query history.
6. Commit the four SQL files with a clear commit message, e.g. `"Add Week 1 SQL: setup, filtering, aggregation, joins"`.
7. Push to GitHub and open the repo in your browser to confirm the files render correctly.

---

## Block 5: Documenting Data Quality Notes (45 min)

**Goal:** Capture the messiness you found this week before it's forgotten — this is the difference between a project and a real analysis.

### Do this: 5 tasks

1. Create `notes/data_quality.md`.
2. Write up the unmatched-station issue from Day 4, Block 3: what it is, how many rows it affects, and whether it's worse in older data.
3. Note any other inconsistencies you ran into across the week — schema differences between years, capitalization issues from Day 1, anything else that took extra handling.
4. For each issue, write one sentence on how you handled it (or chose not to) and why.
5. Commit and push `notes/data_quality.md`.

---

## Block 6: Final Polish & Reflection (45 min)

### Do this: 4 tasks

1. Open your repo on GitHub as a stranger would — click through the README and each SQL file. Fix anything confusing or broken.
2. Add a `LICENSE` file if you want the repo to be reusable by others (MIT is a common, simple choice).
3. Double check `.gitignore` actually kept your database file and any large raw data out of the commit history.
4. Confirm the repo is set to public and the link works in a private/incognito browser window.

### Reflection

Answer in a few sentences each:

1. Look at your repo as it stands now versus your actual working process this week — messier queries, dead ends, trial and error. What did packaging it up force you to clean up or decide that you'd left vague while you were just exploring?
2. If a hiring manager opened this repo cold, what's the one thing in it you'd want them to see first — and does the README currently lead them there?
3. Week 2 will add more files to this same repo. What's one thing about today's structure or README that you'd want to establish as a habit now, so it doesn't need fixing retroactively in Week 6?

**Deliverable:** A public GitHub repository containing `README.md`, four SQL files under `sql/`, and `notes/data_quality.md` — pushed and viewable at your repo's URL.