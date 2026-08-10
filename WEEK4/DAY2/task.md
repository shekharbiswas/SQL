# Day 2 Task: Building the First Visuals

**Total time: 6 hours**

Today turns Day 1's model into the first real chunk of the dashboard: a revenue trend, a category breakdown, and a state breakdown, all built on a small set of explicit measures instead of raw fields. By the end of today, three of the dashboard's core visuals should be in place and verified against the SQL that produced the underlying table.

Read `week4_day2_concepts.md` first — this task assumes the revenue-definition decision and the Top N/granularity reasoning from that file.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Explicit DAX measures | 60 min |
| 2 | Revenue trend line chart | 60 min |
| 3 | Category revenue bar chart (Top N) | 75 min |
| 4 | State revenue bar chart | 60 min |
| 5 | Cross-checking against SQL + layout pass | 75 min |
| 6 | Reflection | 30 min |

---

## Block 1: Explicit DAX Measures (60 min)

**Goal:** Build the small set of measures every visual today and tomorrow will be built from.

### Do this: 6 tasks

1. Decide, in writing, whether `Total Revenue` means item price alone or price plus freight — check this against Day 1's scope note if it was already decided there.
2. Write a `Total Revenue` measure using `SUM()` over the appropriate column(s).
3. Write an `Order Count` measure using `DISTINCTCOUNT()` on the order ID, not a plain row count (since the table is at order-item grain, a row count would overcount).
4. Write an `Avg Review Score` measure and an `Avg Delivery Days` measure.
5. Add all four measures to a visible measures table or clearly labeled section of the Fields pane so they're easy to find tomorrow.
6. Sanity-check each measure with a quick card visual against a known total, then delete the card once confirmed.

---

## Block 2: Revenue Trend Line Chart (60 min)

**Goal:** A trend chart that's honest about Olist's actual order volume over time.

### Do this: 5 tasks

1. Build a line chart with order date on the axis and `Total Revenue` as the value, at daily grain first, just to see the raw shape.
2. Decide, based on what that first version shows, whether to switch to monthly grain or trim the sparse early months — make the call from Day 2 Concepts, not by default.
3. Rebuild the chart at the chosen grain.
4. Add a secondary line or a tooltip showing `Order Count` alongside revenue, so a revenue dip can be read against volume, not just eyeballed.
5. Title the chart with what it actually shows (e.g., "Monthly Revenue, Jan 2017–Aug 2018") rather than a generic label.

---

## Block 3: Category Revenue Bar Chart — Top N (75 min)

**Goal:** A readable category breakdown, not a 70-bar wall.

### Do this: 6 tasks

1. Build a first-pass bar chart of `Total Revenue` by product category, just to see the full distribution.
2. Apply a Top N filter (Power BI's built-in Top N, or a DAX rank measure) to cut it down to a genuinely readable number of categories.
3. Decide whether to add an "Other" bucket summing everything outside the Top N, and build it if so.
4. Sort the chart by revenue, not alphabetically.
5. Add `Order Count` or `Avg Review Score` as a tooltip field, so a category's story isn't reduced to revenue alone.
6. Title the chart clearly, including how many categories are shown (e.g., "Top 10 Categories by Revenue").

---

## Block 4: State Revenue Bar Chart (60 min)

**Goal:** A state-level view, with the same cardinality discipline as the category chart.

### Do this: 5 tasks

1. Build a bar chart of `Total Revenue` by customer state.
2. Decide whether all states are readable as-is or whether this chart also needs a Top N treatment — 27 is fewer than 70, but check rather than assume.
3. Sort by revenue and confirm state abbreviations or names are clean and consistent (Olist uses two-letter state codes; confirm they're not being read as a numeric or sorted oddly).
4. Add `Order Count` as a tooltip field.
5. Title the chart clearly.

---

## Block 5: Cross-Checking Against SQL + Layout Pass (75 min)

**Goal:** Confirm every number on the page traces back to something verifiable, then arrange the three visuals into a coherent first layout.

### Do this: 6 tasks

1. Pick the top category from Block 3's chart and run the equivalent `SUM`/`GROUP BY` query directly against Day 1's summary table in DuckDB — confirm the numbers match exactly.
2. Do the same spot-check for one state from Block 4's chart.
3. Do the same spot-check for one month's total from Block 2's line chart.
4. If any number doesn't match, trace whether the issue is in the DAX measure, the visual's filter context, or the underlying SQL table — and fix it at the source rather than adjusting the visual to match.
5. Arrange the three visuals into a rough first-pass layout — trend chart prominent, category and state charts below or beside it — without worrying yet about final polish (that's Thursday).
6. Save the `.pbix` file.

---

## Block 6: Reflection (30 min)

Answer in a few sentences each:

1. What did the daily-grain version of the revenue trend chart look like before you adjusted the granularity, and why was the adjusted version the more honest one to present?
2. Which of today's three charts needed the most cardinality management, and what would the chart have looked like if you'd skipped that step?
3. Block 5 asked you to trace any mismatch back to its source rather than patching the visual. Did you find one today, and if so, where did it actually originate?
4. Tomorrow adds slicers, cross-filtering, and a delivery-time-vs-review-score visual. Based on today's measures and table structure, what do you expect to be straightforward, and what do you expect to need real thought?

**Deliverable:** A `.pbix` file with four explicit measures (`Total Revenue`, `Order Count`, `Avg Review Score`, `Avg Delivery Days`), a revenue trend line chart, a category revenue bar chart (Top N), and a state revenue bar chart — all cross-checked against the Day 1 SQL summary table, arranged in a rough first-pass layout.