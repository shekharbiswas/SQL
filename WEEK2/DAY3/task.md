# Day 3: NULL Handling, COALESCE & CASE WHEN

**Total time: 6 hours**

The unmatched-station problem has been sitting in your notes since Week 1, Day 4 — worked around, never actually resolved. Today you go back and handle it deliberately, using `COALESCE` and `CASE WHEN`, and apply the same tools to the other inconsistencies you've been noting since then: rider type variants, missing durations, and anything else that's been quietly making your aggregates less trustworthy than they look.

Read `day3_concepts.md` before starting if you haven't already — this task file assumes you understand three-valued logic, `IS NULL` vs `= NULL`, and how `CASE WHEN` behaves inside an aggregate.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | NULL semantics and three-valued logic | 60 min |
| 2 | COALESCE for defaults | 75 min |
| 3 | CASE WHEN for categorization | 90 min |
| 4 | Cleaning the Week 1 station gap, with CTEs | 75 min |
| 5 | Reflection | 60 min |

---

## Block 1: NULL Semantics and Three-Valued Logic (60 min)

**Goal:** Confirm, with real queries against real data, how `NULL` actually behaves — not how it seems like it should behave.

### Do this: 6 tasks

1. Run `SELECT COUNT(*) FROM trips` and `SELECT COUNT(duration) FROM trips` side by side. If the numbers differ, you have `NULL` durations — note how many.
2. Try filtering `WHERE duration = NULL` and confirm it returns nothing, even though `NULL` durations exist. Then rewrite it correctly with `IS NULL`.
3. Find every column in `trips` and `stations` that can contain `NULL`, by running `IS NULL` counts on each candidate column. Keep a running list.
4. Revisit your Week 1, Day 4, Block 3 `LEFT JOIN` between `trips` and `stations`. Confirm again how many rows have `NULL` in a `stations` column, and cross-check it against your notes from that day.
5. Write a query using `NOT IN` against a column you now know can contain `NULL` (from Task 3), and confirm whether it silently returns zero or fewer rows than expected — this is the Day 1 pitfall, now demonstrated on purpose.
6. Rewrite Task 5 using `NOT EXISTS` with a correlated subquery, and confirm it returns the correct, non-empty result.

---

## Block 2: COALESCE for Defaults (75 min)

**Goal:** Replace missing values with deliberate, visible defaults.

### Do this: 7 tasks

1. Write a query selecting `neighborhood` from a joined `trips`/`stations` result, wrapped in `COALESCE(neighborhood, 'Unknown')`, and confirm rows that previously showed `NULL` now show `'Unknown'`.
2. Group that query by the `COALESCE`d neighborhood and count rides per group. Confirm `'Unknown'` now appears as its own visible group instead of being dropped or hidden.
3. Compare the total row count of this grouped result against a version that grouped by raw `neighborhood` without `COALESCE`. Confirm the `NULL` group's rides are now accounted for somewhere instead of vanishing.
4. Apply `COALESCE` to any duration-related `NULL`s you found in Block 1, using a reasonable fallback (for example, the overall average duration, computed with a scalar subquery from Day 1). Justify the fallback choice in one sentence.
5. Try chaining three arguments in a single `COALESCE` on a column of your choice, even if the third argument is just a hardcoded default like `0` or `'Unknown'`.
6. If any text columns contain placeholder values like `'N/A'` or empty strings instead of real `NULL`s, use `NULLIF` to convert them to actual `NULL`s first, then `COALESCE` them to your chosen default.
7. Write one sentence on a case where you'd deliberately choose *not* to `COALESCE` a `NULL` — where leaving it as `NULL`, and filtering it out explicitly, is more honest than papering over it with a default.

---

## Block 3: CASE WHEN for Categorization (90 min)

**Goal:** Build your own categories and conditional counts directly in SQL.

### Do this: 8 tasks

1. Write a `CASE WHEN` that buckets stations into capacity tiers (for example, Small / Medium / Large), with an `ELSE` catching anything unmatched, including `NULL` capacity.
2. Group stations by that tier and count how many fall into each — including the `ELSE` bucket.
3. Write a `CASE WHEN` that buckets trip duration into ranges (for example, under 10 min, 10–30 min, over 30 min), and count trips per bucket.
4. Use `COUNT(CASE WHEN ... THEN 1 END)` to compute, in a single query, the number of casual rides and member rides per neighborhood — no separate filtered queries, no `GROUP BY` on rider type itself.
5. Extend Task 4 to add a third conditional count for any rider type value that doesn't cleanly match either of the two you expected — this surfaces inconsistent or unexpected values in the column.
6. Investigate what Task 5 actually turned up. Are there genuinely a third rider type, inconsistent capitalization, or something else? Note it.
7. Write a `CASE WHEN` that labels each trip as `'Weekday'` or `'Weekend'` based on its start date, and use it to compare ride counts between the two.
8. Combine two `CASE WHEN` expressions in one query — capacity tier and duration bucket, for example — to build a small cross-tabulated summary in a single `SELECT`.

---

## Block 4: Cleaning the Week 1 Station Gap, With CTEs (75 min)

**Goal:** Actually resolve the unmatched-station problem, using Day 2's CTEs alongside today's `COALESCE` and `CASE WHEN`.

### Do this: 6 tasks

1. Build a CTE that joins `trips` to `stations` with a `LEFT JOIN`, and applies `COALESCE(neighborhood, 'Unknown')` and `COALESCE(capacity, 0)` in the same step.
2. Build a second CTE on top of the first that adds a `CASE WHEN` column flagging each row as `'Matched'` or `'Unmatched'`, based on whether the original `stations` join produced a real value or a default.
3. Use the two CTEs together to report total rides and average duration, split by the `'Matched'`/`'Unmatched'` flag — this quantifies, precisely, how much of your data the station gap actually affects.
4. Decide whether `stations_summary` from Week 1 should be rebuilt using this cleaner approach. Rebuild it if so, using `CREATE TABLE stations_summary AS SELECT ...` from the chained CTEs.
5. Update `notes/data_quality.md` from your Week 1 repo with what you found today: the exact percentage of rides affected by the unmatched-station gap, and how you chose to handle it going forward.
6. Commit the updated `stations_summary` (if rebuilt) and the updated notes file to your GitHub repo from Week 1, Day 5.

---

## Block 5: Reflection (60 min)

Answer in a few sentences each:

1. Block 1 demonstrated the `NOT IN`/`NULL` pitfall on real data from this dataset, not a hypothetical. Where else, now that you know which columns can contain `NULL` from your Task 3 list, would you go back and double check earlier Week 1 or Week 2 queries for the same silent failure?
2. Block 2, Task 7 asked you to name a case where `NULL` should stay `NULL` rather than get a default. Explain that choice in more depth now — what would `COALESCE`ing it have hidden, and from whom?
3. Block 3 surfaced whether `rider_type` actually has more than two clean values. If it does, walk through how you'd decide whether that's a data entry problem, a genuine third category, or something that changed partway through the seven-year date range.
4. Block 4 quantified exactly how much of your data the station gap affects. Now that it's a real percentage instead of a vague concern, has your opinion changed on whether it's a small footnote or something that should be flagged prominently in any report built on this data?
5. Tomorrow moves into detecting anomalous periods in the trip data over time — separating a genuine trend from a data gap or a one-off event. Based on everything you've cleaned and flagged today, name one specific period or pattern in the data you'd want to investigate first, and why.

**Deliverable:** The two chained CTEs from Block 4 (joined-and-cleaned trips, plus the matched/unmatched flag), the updated `stations_summary` if rebuilt, and the updated `notes/data_quality.md`, all committed to your Week 1 GitHub repo.