# Day 5: Consolidating Week 2 in Your Repo

**Total time: 6 hours**

Week 2 produced a lot that's currently scattered across four days of query history: subqueries, CTEs, a cleaned `stations_summary`, an anomaly-detection pipeline, and `notes/week2_trend_analysis.md` from Day 4. Some of it is already committed — Day 3 and Day 4 both had you push updates — but the repo as a whole hasn't been reviewed as one piece since Week 1, Day 5. Today is that review: organizing Week 2's files properly, updating the top-level README so it actually reflects two weeks of work instead of one, and closing out the week before Week 3 introduces window functions and dbt.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Reviewing and organizing Week 2's SQL | 75 min |
| 2 | Updating the repo structure | 60 min |
| 3 | Updating the top-level README | 75 min |
| 4 | Consolidating data quality notes | 60 min |
| 5 | Final review | 45 min |
| 6 | Reflection | 45 min |

---

## Block 1: Reviewing and Organizing Week 2's SQL (75 min)

**Goal:** Get every working query from this week into a clean, committed file — not left in query history.

### Do this: 7 tasks

1. Go back through Day 1–4 and identify the final, working version of each day's key deliverables: the subquery exercises, the chained CTEs, the NULL-cleaning CTEs, and the anomaly-detection pipeline.
2. Create four files under `sql/` for this week — `day1_subqueries.sql`, `day2_ctes.sql`, `day3_null_handling.sql`, `day4_anomaly_detection.sql` — following the same naming pattern as Week 1's files.
3. Copy each day's finished, working queries into the matching file, with a one-line comment above each explaining what it answers — the same convention from Week 1, Day 5.
4. Re-run every query in all four files against your live database to confirm they still work exactly as written, including the rebuilt `stations_summary` from Day 3.
5. Check whether any query references a table or column that no longer exists or was renamed since you first wrote it, and fix any that break.
6. Remove or rewrite any query that was clearly a dead end or an intentionally broken example (like the Day 1 scalar-subquery error task) — keep only what's meant to stand as a working reference.
7. Commit the four new SQL files with a clear message, e.g. `"Add Week 2 SQL: subqueries, CTEs, NULL handling, anomaly detection"`.

---

## Block 2: Updating the Repo Structure (60 min)

**Goal:** Confirm the structure set up in Week 1 actually scales the way you predicted it would.

### Do this: 6 tasks

1. Look at your full `sql/` folder now that it holds eight files across two weeks. Is it still easy to tell, at a glance, what week and day each file belongs to?
2. If the flat file-per-day structure is starting to feel cluttered, consider whether subfolders (`sql/week1/`, `sql/week2/`) would help — and if you make that change, update any file paths referenced elsewhere in the repo.
3. Confirm `notes/` is similarly organized — you should now have `data_quality.md` and `week2_trend_analysis.md` in there, plus anything else you've written along the way.
4. Recall your answer to Week 1, Day 5, Block 2, Task 6, where you predicted whether the structure would hold up. Write one sentence on whether it actually did.
5. Confirm `.gitignore` is still correctly excluding your local database file and hasn't accidentally let anything large slip into a commit over the past two weeks.
6. Commit any structural changes from this block before moving on, so the reorganization is captured cleanly in Git history.

---

## Block 3: Updating the Top-Level README (75 min)

**Goal:** Bring the README up to date so it represents the project as it actually stands — two weeks in, not one.

### Do this: 8 tasks

1. Add a "Week 2" section to the README, parallel in structure to the existing Week 1 content — what was built, what tables or files resulted, and what questions it answers.
2. Update the "What's in `sql/`" section to include all four new Week 2 files, with the same one-line-per-file style used for Week 1.
3. Update the "Known Data Issues" section — the unmatched-station gap is no longer just noted, it's been quantified and partially resolved as of Day 3. Rewrite that section to reflect the current state, not the Week 1 state.
4. Add a short "Key Findings" entry summarizing the trend or anomaly investigated in Day 4's mini-project, without copying the full write-up — link to `notes/week2_trend_analysis.md` for the details instead.
5. Update the "Next Steps" section to reflect what's actually coming next: Week 3's advanced SQL and dbt introduction, rather than the general placeholder written in Week 1.
6. Re-read the full README from the top, as if seeing the project for the first time. Confirm nothing referencing "one week" or "just getting started" is left over from the original Week 1, Day 5 version.
7. Confirm every internal link in the README (to `notes/data_quality.md`, `notes/week2_trend_analysis.md`, and any SQL files) actually resolves correctly.
8. Commit the updated README with a clear message, e.g. `"Update README for Week 2"`.

---

## Block 4: Consolidating Data Quality Notes (60 min)

**Goal:** Make sure two weeks of scattered observations read as one coherent record, not a pile of separate entries.

### Do this: 6 tasks

1. Re-read `notes/data_quality.md` from start to finish. Identify anything written in Week 1 that Day 3 of this week has since resolved or changed.
2. Update those entries directly rather than just appending new ones underneath — a reader should get the current, accurate picture, not a changelog they have to reconstruct themselves.
3. Add a short section (if not already there from Day 3) explicitly stating the final, quantified impact of the unmatched-station gap, and how it's now handled in `stations_summary`.
4. Cross-reference `notes/week2_trend_analysis.md` from the data quality file — if the anomaly investigation surfaced anything relevant to data quality (rather than just ridership trends), note that connection in both directions.
5. Read both notes files together and check for any contradictions between what Week 1 assumed and what Week 2 actually found. Resolve any you find.
6. Commit the consolidated notes.

---

## Block 5: Final Review (45 min)

### Do this: 5 tasks

1. Open the repo fresh, as a stranger would, starting from the README. Click through every section and every linked file.
2. Confirm the repo tells a coherent story end to end: Week 1's foundational build, Week 2's advanced querying and data cleaning, and a clear pointer to what's next.
3. Check that `stations_summary` in your live database matches what the committed SQL would actually produce if run fresh — confirm there's no drift between what's documented and what's real.
4. Fix any broken links, outdated language, or inconsistent formatting you find during this pass.
5. Push all final changes and confirm the repo renders correctly on GitHub.

---

## Block 6: Reflection (45 min)

Answer in a few sentences each:

1. Compare how long today's consolidation took against how long you expected. Was two weeks of work harder or easier to bring together than you assumed going in?
2. Block 3 asked you to rewrite the "Known Data Issues" section rather than just append to it. What does keeping documentation *current*, instead of just *additive*, actually require — and is that something you'll keep doing naturally going forward, or does it need to stay a deliberate step like today?
3. Looking at the repo as a whole now, what's the single query or table you're most confident you could defend to a skeptical reviewer, start to finish — and what's the one you'd still want to double check before standing behind it?
4. Week 3 moves into window functions and a first introduction to dbt. Based on the repo as it stands today, what's one thing about your current SQL files or structure that you expect dbt's model-based approach will make you rethink?

**Deliverable:** An updated top-level `README.md` covering both weeks, four new SQL files under `sql/` for Week 2, consolidated notes in `notes/`, and a clean, fully pushed GitHub repo ready to build on in Week 3.